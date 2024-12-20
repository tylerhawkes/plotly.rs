<div align="center">
  <h1>Plotly.rs</h1>
  <p><strong>Plotly for Rust</strong></p>
  <p>
    <a href="https://github.com/plotly/plotly.rs/actions?query=branch%3Amain" style="text-decoration: none!important;">
        <img src="https://img.shields.io/github/actions/workflow/status/plotly/plotly.rs/ci.yml?branch=main" alt="Build status">
    </a>
    <a href="https://crates.io/crates/plotly" style="text-decoration: none!important;">
        <img src="https://img.shields.io/crates/v/plotly.svg" alt="Crates.io">
    </a>
    <a href="https://crates.io/crates/plotly" style="text-decoration: none!important;">
        <img src="https://img.shields.io/crates/d/plotly" alt="Downloads">
    </a>
	<a href="https://docs.rs/plotly" style="text-decoration: none!important;">
        <img src="https://img.shields.io/badge/docs.rs-plotly-green" alt="Documentation">
    </a>
	<a href="https://app.codecov.io/github/plotly/plotly.rs" style="text-decoration: none!important;">
        <img src="https://img.shields.io/codecov/c/github/igiagkiozis/plotly" alt="Code coverage">
    </a>
  </p>
  <h4>
    <a href="https://plotly.github.io/plotly.rs/content/getting_started.html">Getting Started</a>
    <span> | </span>
    <a href="https://plotly.github.io/plotly.rs/content/recipes.html">Recipes</a>
    <span> | </span>
    <a href="https://docs.rs/crate/plotly/">API Docs</a>
    <span> | </span>
    <a href="https://github.com/plotly/plotly.rs/tree/main/CHANGELOG.md">Changelog</a>
  </h4>
</div>

<div align="center">
  <a href="https://dash.plotly.com/project-maintenance">
    <img src="https://dash.plotly.com/assets/images/maintained-by-community.png" width="400px" alt="Maintained by the Plotly Community">
  </a>
</div>

# Table of Contents

* [Introduction](#introduction)
* [Basic Usage](#basic-usage)
    * [Exporting an Interactive Plot](#exporting-an-interactive-plot)
    * [Exporting a Static Image](#exporting-a-static-image)
    * [Usage Within a Wasm Environment](#usage-within-a-wasm-environment)
* [Crate Feature Flags](#crate-feature-flags)
* [Contributing](#contributing)
* [License](#license)

# Introduction

A plotting library for Rust powered by [Plotly.js](https://plot.ly/javascript/).

Documentation and numerous interactive examples are available in the [Plotly.rs Book](https://plotly.github.io/plotly.rs/content/getting_started.html), the [examples/](https://github.com/plotly/plotly.rs/tree/main/examples) directory and [docs.rs](https://docs.rs/crate/plotly).


For changes since the last version, please consult the [changelog](https://github.com/plotly/plotly.rs/tree/main/CHANGELOG.md).

# Basic Usage

Add this to your `Cargo.toml`:

```toml
[dependencies]
plotly = "0.11"
```

## Exporting a single Interactive Plot

Any figure can be saved as an HTML file using the `Plot.write_html()` method. These HTML files can be opened in any web browser to access the fully interactive figure.

```rust
use plotly::{Plot, Scatter};

let mut plot = Plot::new();
let trace = Scatter::new(vec![0, 1, 2], vec![2, 1, 0]);
plot.add_trace(trace);

plot.write_html("out.html");
```

By default, the Plotly JavaScript library and some [MathJax](https://docs.mathjax.org/en/latest/web/components/index.html) components will always be included via CDN, which results in smaller file-size, but slightly slower first load as the JavaScript libraries have to be downloaded first. Alternatively, to embed the JavaScript libraries (several megabytes in size) directly into the HTML file, `plotly-rs` must be compiled with the feature flag `plotly_embed_js`. With this feature flag the Plotly and MathJax JavaScript libraries are directly embedded in the generated HTML file. It is still possible to use the CDN version, by using the `use_cdn_js` method.

```rust
// <-- Create a `Plot` -->

plot.use_cdn_js();
plot.write_html("out.html");
```

If you only want to view the plot in the browser quickly, use the `Plot.show()` method.

```rust
// <-- Create a `Plot` -->

plot.show(); // The default web browser will open, displaying an interactive plot
```

## Exporting a Static Image

To save a plot as a static image, the `kaleido` feature is required:

```toml
# Cargo.toml

[dependencies]
plotly = { version = "0.11", features = ["kaleido"] }
```

With this feature enabled, plots can be saved as any of `png`, `jpeg`, `webp`, `svg`, `pdf` and `eps`. Note that the plot will be a static image, i.e. they will be non-interactive.

Exporting a simple plot looks like this:

```rust
use plotly::{ImageFormat, Plot};

let mut plot = Plot::new();
let trace = Scatter::new(vec![0, 1, 2], vec![2, 1, 0]);
plot.add_trace(trace);

plot.write_image("out.png", ImageFormat::PNG, 800, 600, 1.0);
```

### _Kaleido dependency_

On your host, when building this project with the `kaleido` feature enabled the Kaleido binary is downloaded automatically for your system's architecture at compile time from the official Kaleido [release page](https://github.com/plotly/Kaleido/releases). This library currently supports `x86_64` on Linux and Windows, and both `x86_64` and `aarch64` on macOS.

When building application for other targets that depend on this feature, the `Kaleido` binary will need to be installed manually on the target machine. Currently, the location where the binary is expected is hardcoded depending on the target OS. E.g., on Linux this defaults to `~/.config/kaleido`. This is defined in source code at [here](https://github.com/plotly/plotly.rs/blob/1405731b5121c1343b491e307222a21ef4becc5e/plotly_kaleido/src/lib.rs#L89)

## Usage Within a Wasm Environment

Using `Plotly.rs` in a Wasm-based frontend framework is possible by enabling the `wasm` feature:

```toml
# Cargo.toml

[dependencies]
plotly = { version = "0.11", features = ["wasm"] }
```

First, make sure that you have the Plotly JavaScript library in your base HTML template:

```html
 <!-- index.html -->

<!doctype html>
<html lang="en">
    <head>
        <!-- snip -->
        <script src="https://cdn.plot.ly/plotly-2.14.0.min.js"></script>
    </head>
    <!-- snip -->
</html>
```

A simple `Plot` component would look as follows, using `Yew` as an example frontend framework:

```rust
use plotly::{Plot, Scatter};
use yew::prelude::*;


#[function_component(PlotComponent)]
pub fn plot_component() -> Html {
    let p = yew_hooks::use_async::<_, _, ()>({
        let id = "plot-div";
        let mut plot = Plot::new();
        let trace = Scatter::new(vec![0, 1, 2], vec![2, 1, 0]);
        plot.add_trace(trace);

        async move {
            plotly::bindings::new_plot(id, &plot).await;
            Ok(())
        }
    });


        use_effect_with_deps(move |_| {
            p.run();
            || ()
        }, (),
    );


    html! {
        <div id="plot-div"></div>
    }
}
```

More detailed standalone examples can be found in the [examples/](https://github.com/plotly/plotly.rs/tree/main/examples) directory.

# Crate Feature Flags

The following feature flags are available:

### `kaleido`

Adds plot save functionality to the following formats: `png`, `jpeg`, `webp`, `svg`, `pdf` and `eps`.

### `plotly_image`

Adds trait implementations so that `image::RgbImage` and `image::RgbaImage` can be used more directly with the `plotly::Image` trace.

### `plotly_ndarray`

Adds support for creating plots directly using [ndarray](https://github.com/rust-ndarray/ndarray) types.

### `plotly_embed_js`

By default, the CDN version of `plotly.js` is used in the library and in the generated HTML files. This feature can be used to opt in for embedding `plotly.min.js` in the generated HTML files. The benefit is that the plot will load faster in the browser.

However, there are two downsides of using this feature flag, one is that the resulting html will be much larger, as a copy of the `plotly.min.js` library is embedded in each HTML file. The second, more relevant, is that a copy of the `plotly.min.js` library needs to be compiled in the `plotly-rs` library itself which increases the size by approx `3.5 Mb`.

When the feature is enabled, users can still opt in for the CDN version by using the method `use_cdn_js`.

Note that when using `Plot::to_inline_html()`, it is assumed that the `plotly.js` library is already in scope within the HTML file, so enabling this feature flag will have no effect.

### `wasm`

Enables compilation for the `wasm32-unknown-unknown` target and provides access to a `bindings` module containing wrappers around functions exported by the plotly.js library.

# Contributing

* If you've spotted a bug or would like to see a new feature, please submit an issue on the [issue tracker](https://github.com/plotly/plotly.rs/issues).

* Pull requests are welcome, see the [contributing guide](https://github.com/plotly/plotly.rs/tree/main/CONTRIBUTING.md) for more information.

# License

`Plotly.rs` is distributed under the terms of the MIT license.

See [LICENSE-MIT](https://github.com/plotly/plotly.rs/tree/main/LICENSE-MIT), and [COPYRIGHT](https://github.com/plotly/plotly.rs/tree/main/COPYRIGHT) for details.
