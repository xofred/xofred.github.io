---
title:  "Setup Rails 5 Webpacker on silicon MacBook"
tags: webpack node mac legacy

---

*The new MacBook seems to hate the legacy project LOL*

If the new macOS version installs `node-sass` with an error that `python2` cannot be found, see [here](https://stackoverflow.com/questions/71591971/how-to-fix-zsh-command-not-found-python-error-macos-monterey-12-3-python) to install `python2` first.

If still getting a bunch of compilation errors, after `node-sass` is installed. Check if the `node` and `yarn` versions are too high. For example in our `yarn.lock`, it is `node-sass "^4.12.0"`, which, according to here, corresponds to the highest version of `node` 12. As for `yarn`, it corresponds to [version 1.x](https://classic.yarnpkg.com/en/docs/install#mac-stable).

[The lower version of `node` on the m1 MacBook may have problems running `webpack`](https://github.com/pmmmwh/react-refresh-webpack-plugin/issues/259). We can solve it by [installing the x86-compatible version](https://stackoverflow.com/questions/65342769/install-node-on-m1-mac).

My environment:
- macOS 12.4 (21F79) Apple M1 Pro
- ruby 2.6.5p114 (2019-10-01 revision 67812) [-darwin21]
- Rails 5.2.8
- node v12.22.12
- yarn 1.22.19
- @rails/webpacker@^4.0.7
- node-sass "^4.12.0"

### Reference

[How to fix "zsh: command not found: python" error? (macOS Monterey 12.3, python 3.10, Atom IDE, atom-python-run 0.9.7)](https://stackoverflow.com/questions/71591971/how-to-fix-zsh-command-not-found-python-error-macos-monterey-12-3-python)

[Node.js: Python not found exception due to node-sass and node-gyp](https://stackoverflow.com/questions/45801457/node-js-python-not-found-exception-due-to-node-sass-and-node-gyp)

[Installation \| Yarn](https://classic.yarnpkg.com/en/docs/install#mac-stable)

[Apple Silicon / arm64 / M1 compatibility? · Issue #259 · pmmmwh/react-refresh-webpack-plugin](https://github.com/pmmmwh/react-refresh-webpack-plugin/issues/259)

[Install Node on M1 Mac](https://stackoverflow.com/questions/65342769/install-node-on-m1-mac)
