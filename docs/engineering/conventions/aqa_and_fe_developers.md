# AQA & FE Developers

QA Locators - is attributes which gonna help for automation testing.
Injecting QA Locators should increase integration time between AutoQA and FE teams.

## Overview

Supported data attributes:

- `[test-id]` - Unique element identifier;
- `[test-target]` - Element with interactions inside components;
- `[test-loading]` - Loading state of element;


## [test-id]

- Overview: **Name for container element**
- Directive selector: `mmTestId`
- Value: `[PAGE_NAME].[COMPONENT_NAME]`
- Example: PRODUCT_DETAILS_PAGE.QUANTITY_PICKER
- Description: This attribute must be added to main blocks of business logic (Modal window, Quantity picker, Paginates and etc).
- Usage: 
    - `form`
    - `table`
    - `sidebar`
    - `ul/ol`
    - `containers`
    
    
## [test-target]
- Overview: **Name for element**
- Directive selector: `mmTestTarget`
- Example: increase-button, decrease-button, quantity-input
- Description: Attribute must be placed inside of main block of business logic for easy bind to elements with interactions and action with it
- Usage:
    - `checkbox`
    - `radio button`
    - `pagination`
    - `h1-h5`
    - `drag-n-drop` area
    
## [test-loading]
- Overview: **Loading state of element**
- Directive selector: `mmTestLoading`
- Example: load, ready
- Description: Attribute for element with loading state, which use spinners or overlay in loading time
- Usage:
    - `table`
    - `accordion`
    - `buttons`in form
