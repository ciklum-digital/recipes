# Angular 8+ (Ivy)

## Asynchronous components

Ivy compiler is currently experimental and can be enabled by providing compiler option. Let's edit our `tsconfig.app.json`:

```json
{
  "extends": "./tsconfig.json",
  "include": [
    "src/**/*.ts"
  ],
  "exclude": [
    "src/test.ts",
    "src/**/*.spec.ts"
  ],
  "angularCompilerOptions": {
    "enableIvy": true
  }
}
```

In simple terms Ivy is a compiler that provides a completely new pipeline.

This compiler also allows us to load components asynchronously on demand and this component is not linked with any module.

Let's imagine that we have some blog, there are 100 articles. When you click on the article - it reveals and you see the full content. Let's create our asynchronous `ArticleComponent`:

```typescript
@Component({
  selector: 'app-article',
  template: `
    <div [innerHTML]="content"></div>
  `,
  styleUrls: ['./article.component.scss'],
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class ArticleComponent {
  public content: SafeHtml | null = null;
 
  constructor(private ref: ChangeDetectorRef, private sanitizer: DomSanitizer) {}
 
  public setContent(content: string): void {
    this.content = this.sanitizer.bypassSecurityTrustHtml(content);
    this.ref.detectChanges();
  }
}
```

Asynchronous components do not have a life cycle, therefore the `ngOnInit`, `ngAfterViewInit`, `ngOnDestroy` methods will not work.

Therefore, in order to perform any cleanup - I would like to have a common interface that asynchronous components can implement and also follow the "dispose" pattern (rough example):

```typescript
interface Disposable {
  dispose(): void;
}
 
export class AsynchronousComponentThatMightHaveSideEffects implements Disposable {
  private readonly destroy$ = new Subject<void>();
 
  constructor(private zone: NgZone) {}
 
  public setup(): void {
    this.zone.runOutsideAngular(() => this.setupEventListener());
  }
 
  public dispose(): void {
    this.destroy$.next();
    this.destroy$.complete();
  }
 
  private setupEventListener(): void {
    fromEvent().pipe(takeUntil(this.destroy$)).subscribe(...);
  }
}
```

Returning to the `ArticleComponent` component, how can we load and render it asynchronously? Very simply, we have a dynamic import:

```typescript
import { Component, ɵrenderComponent as renderComponent, Injector, ChangeDetectionStrategy, OnDestroy } from '@angular/core';
 
@Component({
  selector: 'app-articles',
  template: `
    <div *ngFor="let article of articles; index as index">
      ...
      <button (click)="openArticle(article, index)">Open article</button>
      <app-article class="article-{{ index }}"></app-article>
    </div>
  `,
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class ArticlesComponent implements OnDestroy {
  public articles: Article[] = [];
 
  /**
   * Dynamic import can also be used as a type definition for asynchronous modules
   */
  private articleComponent: import('./article.component').ArticleComponent | null = null;
 
  constructor(private injector: Injector) {}
 
  public ngOnDestroy(): void {
    this.destroyArticleComponent();
  }
 
  public async openArticle({ content }: Article, index: number): Promise<void> {
    this.destroyArticleComponent();
 
    const { ArticleComponent } = await import('./article.component');
 
    this.articleComponent = renderComponent(ArticleComponent, {
      host: `.article-${index}`,
      injector: this.injector
    });
 
    this.articleComponent.setContent(content);
  }
 
  private destroyArticleComponent(): void {
    if (this.articleComponent) {
      this.articleComponent.dispose();
      this.articleComponent = null;
    }
  }
}
```

`ɵrenderComponent` is available to use right now, NOTICE this function works only when the Ivy compiler is enabled. You also need to manually control the life cycle of this component, and re-render it yourself.
