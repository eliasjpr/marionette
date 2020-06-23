# Marionette

Marionette is a one-size-fits-all approach to WebDriver adapters. It works with most all web driver implementations, including:

- [x] Chrome
- [x] Chromium
- [x] Firefox
- [x] Safari
- [x] Edge
- [x] Internet Explorer
- [x] Opera
- [x] PhantomJS
- [x] Webkit GTK
- [x] WPE Webkit
- [x] Android

## Installation

1. Make sure you have Crystal installed. This is a Crystal project and Crystal is required for usage. If you don't have it installed, see https://crystal-lang.org.

2. Add Marionette to an existing project by adding the dependency to your `shard.yml`

   ```yml
   dependencies:
     marionette:
       github: watzon/marionette
       branch: master
   ```

3. Run `shards install` to download and install Marionette as a dependency.

4. Download and have installed at least one [WebDriver](https://www.w3.org/TR/webdriver/). See the [#webdriver](#WebDriver) section below for links to various downloads.

## WebDriver

WebDriver is a protocol which allows browser implementations to be remote controlled via a common interface. It's because of this functionality that frameworks like Marionette are possible. To use the protocol you first have to have installed one of the many WebDriver implementations, here are some of those:

### Firefox

GeckoDriver is implemented and supported by Mozilla directly.

- [Downloads](https://github.com/mozilla/geckodriver/releases)
- [Documentation](https://firefox-source-docs.mozilla.org/testing/geckodriver/Support.html)

### Chrome

ChromeDriver is implemented and supported by the Chromium Project.

- [Downloads](https://sites.google.com/a/chromium.org/chromedriver/downloads)
- [Documentation](https://sites.google.com/a/chromium.org/chromedriver/)

### Opera

OperaChromiumDriver is implemented and supported by Opera Software.


- [Downloads](https://github.com/operasoftware/operachromiumdriver/releases)
- [Documentation](https://github.com/operasoftware/operachromiumdriver/releases)

### Safari

SafariDriver is implemented and supported directy by Apple. It comes pre-installed with Safari and Safari Technology Preview.

- [Documentation](https://developer.apple.com/documentation/webkit/about_webdriver_for_safari)

### Edge

Microsoft is implementing and maintaining the Microsoft Edge WebDriver.

- [Downloads](https://developer.microsoft.com/en-us/microsoft-edge/tools/webdriver/#downloads)
- [Documentation](https://developer.microsoft.com/en-us/microsoft-edge/tools/webdriver)

### Internet Explorer

Only version 11 is supported, and it requires additional [configuration](https://github.com/SeleniumHQ/selenium/wiki/InternetExplorerDriver#required-configuration).

**Note:** Marionette specific configuration instructions coming soon.

## Getting Started

The goal of Marionette is simplicity, which is why it's written in Crystal. Once you have a webdriver installed and sitting comfortably on your path, using it couldn't be easier:

```crystal
require "marionette"

session = Marionette::WebDriver.create_session(:chrome)

# Navigate to crystal-lang.org
session.navigate("https://crystal-lang.org")

# Start an action chain and perform it
session.perform_actions do
  # Click the "INSTALL" link, offsetting by 25 pixels so we make sure we hit it
  click ".main-actions a:nth-child(1)", 25, 25
end

sleep 5
session.close
```

## Browser Manipulation

As shown above, you can initialize a new driver session for whatever driver you want using `Marionette::WebDriver.create_session`, the first and most important argument to which is `:browser`. Browser can be any of `:chrome`, `:firefox`, `:opera`, `:safari`, `:edge`, `:internet_explorer`, `:webkit_gtk`, `:wpe_webkit`, or `:android`.

If the driver for the chosen browser is installed under its usual name that should be all you need to do, if not you may need to provide the binary location via the `:exe_path` argument. Other notable arguments are:

- `:port` - sets the port you want the driver to listen on
- `:env`  - a hash of environment variables for the driver to be aware of
- `:args` - a string array of arguments to pass to the webdriver process
- `:options` - a JSON compatible structure containing browser options. see [here](./src/marionette/driver_options.cr) for some nice helpers.

### Navigation

#### `#navigate`

The first thing you will want to do after launching a browser is to open your website. This can be achieved in a single line:

```crystal
session.navigate("https://crystal-lang.org")
```

#### `#current_url`

You can read the current URL from the browser’s address bar using:

```crystal
session.current_url
# => https://crystal-lang.org
```

#### `#back`

Pressing the browser’s back button:

```crystal
session.back
```

#### `#forward`

Pressing the browser’s forward button:

```crystal
session.forward
```

#### `#refresh`

Refresh the current page:

```crystal
session.refresh
```

#### `#title`

You can read the current page title from the browser:

```crystal
session.title
# => Crystal | The Crystal Programming Language
```

### Windows and Tabs

WebDriver does not make the distinction between windows and tabs. If your site opens a new tab or window, Marionette will let you work with it using a window handle. Each window has a unique identifier which remains persistent in a single session.

#### `#current_window`

You can get the currently active window using:

```crystal
session.current_window
```

This returns a `Window` instance containing a handle and allowing certain functions to be performed directly on the window instance.

#### `#windows`

You can get an array of all currently opened windows using:

```crystal
session.windows
```

#### `#new_window`

You can create a new window or tab using:

```crystal
session.new_window(:window) # default
session.new_window(:tab)

# Or using the Window object

Marionette::Window.new(:window) # default
Marionette::Window.new(:tab)
```

#### `#switch_to_window`

To interact with other windows you have to switch to them, this can be done with:

```crystal
session.switch_to_window(window)

# Or using the Window object

window.switch
```

#### `#close_window`

When you are finished with a window or tab and it is not the last window or tab open in your browser, you should close it and switch back to the window you were using previously:

```crystal
session.close_window(window)

# Or using the Window object

window.close
```

#### `#close_current_window`

Think of this is a shortcut to `#close_window` but for the currently active window:

```crystal
session.close_current_window
```

#### `#stop`

When you are finished with the browser session you should call `stop`, instead of `close`:

```crystal
session.stop
```

Stop will:
- Close all the windows and tabs associated with that WebDriver session
- Close the browser process
- Close the background driver process

Stop will be automatically closed on process exit.

### Frames and IFrames

Frames are a now deprecated means of building a site layout from multiple documents on the same domain. You are unlikely to work with them unless you are working with an pre HTML5 webapp. Iframes allow the insertion of a document from an entirely different domain, and are still commonly used.

If you need to work with frames or iframes, WebDriver allows you to work with them in the same way. Consider a button within an iframe. If we inspect the element using the browser development tools, we might see the following:

```html
<div id="modal">
  <iframe id="buttonframe" name="myframe"  src="https://watzon.github.io">
   <button>Click here</button>
 </iframe>
</div>
```

If it was not for the iframe we would expect to click on the button using something like:

```crystal
session.find_element!("button").click
```

However, if there are no buttons outside of the iframe, you might instead get a _no such element error_. This happens because Marionette is only aware of the elements in the top level document. To interact with the button, we will need to first switch to the frame, in a similar way to how we switch windows. WebDriver offers three ways of switching to a frame.

#### `#switch_to_frame`

The `switch_to_frame` session method allows us to tell the WebDriver that we want to switch the page context to the given frame/iframe:

```crystal
# Find the element
iframe = session.find_element!("#modal>iframe")

# Switch to the frame
session.switch_to_frame(iframe)

# Now we can click the button
session.find_element!("button").click
```

#### `#switch_to_parent_frame`

If you're in a nested set of frames you can switch back to the parent frame using:

```crystal
session.switch_to_parent_frame
```

#### `#leave_frame`

When you're done inside a frame and want to get back to the normal document context you can use:

```crystal
session.leave_frame
```

## Contributing

1. Fork it ( https://github.com/watzon/marionette/fork )
2. Create your feature branch (git checkout -b my-new-feature)
3. Commit your changes (git commit -am 'Add some feature')
4. Push to the branch (git push origin my-new-feature)
5. Create a new Pull Request

## Contributors

- [watzon](https://github.com/watzon)  - creator, maintainer
