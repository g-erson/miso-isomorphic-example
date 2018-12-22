# Single page isomorphic example
This is a minimal example of Miso's Isomorphic feature.

## About isomorphism
Chances are that you happen to know what the mathematical definition of *isomorphism* is. If you do, forget that definition *right now* and treat it as a new term altogether. That'll be easier.

[Isomorphic Javascript](https://en.wikipedia.org/wiki/Isomorphic_JavaScript) is the idea of having a web application "run" on both client *and* the server. The goal is to make single page javascript applications load faster. In frameworks *without* isomorphism, the server sends an html page with a mostly empty body, along with a big javascript file that fills in the body upon loading. This means that the app user won't see the app until the big javascript file is downloaded and run.

In applications *with* isomorphism, the body of the html page is rendered on the server, by running a minimal part of the app serverside. This way, app users will see a page even before the big javascript file loads. The big javascript file, once loaded, takes over the rendering and adds interactivity.

## Required reading

- [The Miso framework](https://haskell-miso.org/) - The awesome framework of which this is an example.
- [The Elm Architecture](https://guide.elm-lang.org/architecture/) - Miso uses the same architectural concepts. If you're not familiar yet, the linked page should quickly get you up to speed about the core idea.
- [Servant](http://haskell-servant.readthedocs.io/en/stable/) - A Haskell web DSL. Used extensively in this example. Specifically, the introduction and first two sections of the tutorial should cover most of the usage in this example.

## The example explained
This example consists of a server, running the app on port `3003`, and a ghcjs client. Three Haskell files define both:

- [`Common.hs`](common/Common.hs) Contains code used by *both* client and server
- [`client/Main.hs`](client/Main.hs) Defines the client
- [`server/Main.hs`](server/Main.hs) Defines the server

For quick overview, here's which parts of the application are defined where:

| Server         | Client             | Common             |
| -------------  | -------------      | -------------      |
| Top level Html | Update function(s) | `Model` data type  |
| Servant routes | subscriptions      | Initial model      |
|                |                    | `Action` data type |
|                |                    | `View` functions   |
|                |                    | Servant routes     |

Many important parts of the application are common to both client and server. It's more interesting, though, to look at what *isn't* common: The `update` function(s) and subscriptions live clientside only. This means that the server *cannot* add interactivity to the app. It can only render the actual Html page, using the initial model and the `View` functions. Any `Action`s referred to in the `View` functions are ignored by the server.

So basically it renders the initial Html page, sends it off to the client which then adds interactivity.

Note that "Servant routes" is listed both in `Common` and in `Server`. The routes in `Common` are linked to pages in the app itself, in this example just the top level `/` home route. The routes in `Server` include those defined in `Common`, but also define other routes, in our case the `/static/` route which serves the `all.js` of the clientside app.


## Running the example

Using nix:

```bash
cd $(nix-build) && bin/server
```

**Note**: The current working directory is important when running the server. The server won't be able to find the clientside part of the app when running the server from some place *other* than the folder with `bin` and `static`. This will prevent the javascript from loading and make the buttons not work. In other news, that's a great way of looking at the part of the app sent by the server.


## Developing the project

You can work on the different sub-projects (`client`, `common` and `server`) using `nix-shell` and `cabal`.

`common`:

```bash
cd common
nix-shell
cabal build
cabal run common-test
...
exit
```

`client`:

```bash
cd client
nix-shell
cabal build
...
exit
```

`server`:

```bash
cd server
nix-shell
cabal build
cabal run server
...
exit
```

You need to make the client available to the server before running `cabal run server`:

```bash
mkdir static
ln -sf $(find ../client -name all.js) static/
```
