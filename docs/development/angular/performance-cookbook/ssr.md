# Server-side rendering

## Table of contents

* [Pre-rendering with Angular Universal](#pre-rendering-with-angular-universal)
* [Pre-rendering pages in-memory](#pre-rendering-pages-in-memory)

# Pre-rendering with Angular Universal

The server-side rendering is indeed has some features, like:

* SEO (search engine optimization): At the time of writing, SPAs (single-page applications) are harder to index by search engines because the content isn’t available on load time. Therefore, the application is likely to fail on several SEO requirements.
* Initial page load could be faster: Since the application still needs to be bootstrapped after the page is loaded, there is an initial waiting time until the user can use the application. This results in a bad user experience.

Imagine the situation that we have several public static pages that are the most visited, it can be a landing page. Users can visit it from mobile devices at low speed and we don’t want to render this page every time. Solution is - pre-rendering.

Server-side pre-rendering is the process of getting a finished rendered piece of page or rendering it when it is first accessed.

Our user goes to the landing page, we check if this page has already been rendered. No? Render it and save it in the appropriate file, next time we will give the rendered page just from the file without any effort.

Let's look at the below code that shows how `Prerenderer` class could be implemented:

```typescript
const document = fs.readFileSync(
  join(process.cwd(), 'dist/index.html'),
  { encoding: 'utf-8' }
);

const stat = promisify(fs.stat);
const readFile = promisify(fs.readFile);
const writeFile = promisify(fs.writeFile);

export class Prerenderer {
  /**
   * Should be stored somewhere else, stored here
   * for demonstrating purposes
   */
  private urlsToPrerender: ReadonlyArray<string> = ['/', '/about'];

  /**
   * Let's store our pre-rendered pages inside the `prerendered` folder
   */
  private path = join(process.cwd(), 'prerendered');

  constructor() {
    this.setupExitListener();
    this.createPrerenderedFolder();
  }

  public prerender(url: string): Promise<string> {
    if (this.shouldSkip(url)) {
      return this.renderModuleFactory(url);
    }

    return this.getPrerendered(url);
  }

  /**
   * Determines whether URL shouldn't be pre-rendered
   */
  private shouldSkip(url: string): boolean {
    return this.urlsToPrerender.indexOf(url) === -1;
  }

  private async getPrerendered(url: string): Promise<string> {
    const path = join(this.path, `${this.transformIndexUrl(url)}.html`);

    try {
      // No exception was thrown = file exists
      await stat(path);
      // Read its content
      return readFile(path, { encoding: 'utf-8' });
    } catch {
      // File doesn't exist, let's render and save it
      const content = await this.renderModuleFactory(url);
      // Write file in background
      writeFile(path, content);
      return content;
    }
  }

  /**
   * HTML file with invalid name will be created for the `/` URL
   */
  private transformIndexUrl(url: string): string {
    return url === '/' ? 'index' : url;
  }

  private renderModuleFactory(url: string): Promise<string> {
    return renderModuleFactory(AppServerModuleNgFactory, {
      url,
      document,
      extraProviders: [
        provideModuleMap(LAZY_MODULE_MAP)
      ]
    });
  }

  private createPrerenderedFolder(): void {
    if (fs.existsSync(this.path)) {
      return;
    }

    fs.mkdirSync(this.path);
  }

  private setupExitListener(): void {
    process.on('exit', () => {
      // Remove `prerendered` folder if the process exits, this means
      // that server is gonna be restarted if any developer made some changes
      // and we don't wanna receive old content
      fs.unlinkSync(this.path);
    });
  }
}
```

After all this - you just need to create an instance of the class and call its method in some handler, let's look at the `Koa` example:

```typescript
const app = new Koa();
const prerenderer = new Prerenderer();

app.use(async (ctx: Context) => {
  ctx.body = await prerenderer.prerender(ctx.url);
});

app.listen(PORT, () => {
  console.log(`Koa server listening on http://localhost:${PORT}`);
});
```

That's it, nothing complicated.

# Pre-rendering pages in-memory
