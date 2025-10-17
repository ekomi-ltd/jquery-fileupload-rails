### jquery-fileupload-rails API Documentation

This gem wraps the blueimp jQuery File Upload plugin for the Rails asset pipeline and provides a Rack middleware to smooth out the iframe-transport fallback. It exposes a small Ruby API surface and ships the JavaScript plugin assets (including optional AngularJS and jQuery UI integrations).

---

## Ruby API

- **Namespaces**: `JQuery::FileUpload::Rails`
  - `VERSION` (String): Gem version constant.
  - `Engine < ::Rails::Engine`: Rails engine that registers asset paths for Sprockets. No public methods.
  - `Middleware`: Rack middleware that normalizes iframe transport responses used by `jquery.iframe-transport`.

### Middleware

- **Class**: `JQuery::FileUpload::Rails::Middleware`
  - Purpose: When the client uses the iframe fallback transport, browsers require the response to be an HTML document containing the raw response in a `<textarea>`. This middleware wraps the downstream response accordingly and forwards status/type metadata.
  - Public methods (Rack interface):
    - `initialize(app)` – standard Rack middleware initializer.
    - `call(env)` – standard Rack call; returns `[status, headers, body]` or a wrapped body in iframe mode.
  - Behavior:
    - Detects iframe transport via `params['X-Requested-With'] == 'IFrame'`.
    - Forces `Content-Type` header to `text/html` during iframe transport.
    - Wraps the response body as:
      - Left: `<!DOCTYPE html><html><body><textarea data-...>`
      - Right: `</textarea></body></html>`
    - Emits metadata attributes when available: `data-status`, `data-statusText`, `data-type`.

#### Enabling the middleware

Add in a Rails initializer:

```ruby
# config/initializers/jquery_fileupload_middleware.rb
Rails.application.config.middleware.use JQuery::FileUpload::Rails::Middleware
```

### Engine

- **Class**: `JQuery::FileUpload::Rails::Engine < ::Rails::Engine`
  - Autoloads the gem's assets under `app/assets` into the host application's asset pipeline. No configuration options are exposed.

### Version

- **Constant**: `JQuery::FileUpload::Rails::VERSION`
  - Example: `"1.0.0"`

---

## JavaScript API (shipped assets)

The gem vendors the blueimp jQuery File Upload plugin and extensions. You interact with them through the jQuery plugin API and, optionally, the AngularJS module. The following files become available to Sprockets (via `//= require`). See README for complete manifests.

### Core jQuery plugin

- Primary file: `jquery-fileupload/jquery.fileupload`
- jQuery interface: `$(selector).fileupload(optionsOrMethod, [arg])`

Common initialization example:

```javascript
$(function () {
  $('#fileupload').fileupload({
    dataType: 'json',
    url: '/uploads',
    autoUpload: true,
    maxFileSize: 10 * 1024 * 1024, // 10 MB
    acceptFileTypes: /(gif|jpe?g|png)$/i
  })
  .on('fileuploaddone', function (e, data) {
    console.log('Uploaded:', data.result);
  })
  .on('fileuploadfail', function (e, data) {
    console.error('Upload failed:', data.errorThrown || data.textStatus);
  });
});
```

Selected methods (invoked as `$(el).fileupload('method', args)`):
- `option` – get/set plugin options.
- `add` – add files programmatically: `{ files: FileList|Array<File> }`.
- `send` – send a file set immediately: same payload as `add`.
- `process` / `processing` – run the processing pipeline (image/audio/video/validate extensions).
- `progress` – return the current progress state.
- `active` – return number of active uploads.

Selected events (listen via `.on(name, handler)`):
- `fileuploadadd`, `fileuploadsubmit`, `fileuploadsend`
- `fileuploaddone`, `fileuploadfail`, `fileuploadalways`
- `fileuploadprogress`, `fileuploadprogressall`

### Processing and extensions

Optional modules exposed via assets:
- `jquery.fileupload-process` – processing pipeline.
- `jquery.fileupload-image` – client-side image previews/resizing.
- `jquery.fileupload-audio` – audio previews.
- `jquery.fileupload-video` – video previews.
- `jquery.fileupload-validate` – client-side validation.
- `jquery.fileupload-ui` and `jquery.fileupload-jquery-ui` – UI widgets and jQuery UI integration.
- `jquery.iframe-transport` – legacy fallback transport (pairs with the Rack middleware above).

Require what you need in your asset manifest, for example:

```javascript
//= require jquery-fileupload
//= require jquery-fileupload/jquery.fileupload-process
//= require jquery-fileupload/jquery.fileupload-image
```

### AngularJS integration (optional)

- Module: `blueimp.fileupload`
- File: `jquery-fileupload/jquery.fileupload-angular`

Usage example:

```javascript
angular.module('app', ['blueimp.fileupload'])
  .controller('UploadCtrl', function ($scope) {
    $scope.options = { url: '/uploads', dataType: 'json' };
  });
```

Directives (non-exhaustive):
- `file-upload` – attaches the plugin to an element.
- `file-upload-progress`, `file-upload-preview`, `download` – helper directives used by the example UI.

Bridge methods available on the jQuery element in the Angular wrapper include calls to `option`, `add`, `send`, `process`, `processing`, `progress`, `active`.

---

## Rails Integration

### Assets

Add to `app/assets/javascripts/application.js`:

```javascript
//= require jquery
//= require jquery_ujs
//= require jquery-fileupload
```

Optional bundles:

```javascript
//= require jquery-fileupload/basic
//= require jquery-fileupload/basic-plus
//= require jquery-fileupload/jquery-ui
//= require jquery-fileupload/angularjs
```

Stylesheets in `app/assets/stylesheets/application.css` (or `.scss`):

```css
*= require jquery.fileupload
*= require jquery.fileupload-ui
```

Optional noscript styles:

```css
*= require jquery.fileupload-noscript
*= require jquery.fileupload-ui-noscript
```

### Controller endpoints

Your upload endpoint should accept multipart POSTs and return JSON (or a type your client expects). Example Rails controller action:

```ruby
class UploadsController < ApplicationController
  def create
    # Save the uploaded file(s) and return JSON payload expected by the UI
    render json: { files: [{ name: params[:file]&.original_filename, url: url_for('/path') }] }
  end
end
```

When supporting legacy browsers that rely on the iframe transport, enable the middleware to wrap responses as HTML and surface status metadata (see Middleware section).

---

## Compatibility

- Rails: 3.1+ (Sprockets asset pipeline); later versions typically work as the gem only provides assets and a simple middleware.
- jQuery File Upload plugin version shipped: see `README.md` for the exact version packaged.

---

## License

This gem is MIT licensed. See `README.md`.


