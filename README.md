# Rails Frankestein Apps

<img src="https://media1.giphy.com/media/l41YxlUqVzatpICbK/giphy.gif" width=400>

React on Rails? What could go wrong?

There’s more support than ever for newfangled fanciness like node packages, webpack, and react!

#### Maintenance
Your tools are out of date! It’s time to upgrade to the latest versions of `node`, `npm`, and `rails`!

``` bash
$ brew upgrade node # >= 8.2
$ gem update rails # >= 5.1
$ npm install -g npm@latest # >= 5.2
```

## Rails 5.1

#### Adding NPM package support to Ruby on Rails

Want to use node/javascript packages in your rails app? You can do that now, in [rails 5.1](http://edgeguides.rubyonrails.org/5_1_release_notes.html), using [`yarn`](https://yarnpkg.com/en/) (a tool that supplements/compliments `npm`).

``` bash
$ brew install yarn --ignore-dependencies
```

In Rails 5.1, when you generate a rails app you will see a `package.json` file (in addition to your `Gemfile`) in your rails app!

[`yarn` works just like `npm`](https://yarnpkg.com/en/docs/usage), and will replace it. So, for example, instead of `npm install`-ing, you will `yarn install`.

> Just like `Gemfile` has a `Gemfile.lock`, rails 5.1. uses `yarn` and `yarn.lock` to lock in your `package.json` dependencies.

## Webpack on Rails 5.1

#### For an existing rails app (SKIP THIS STEP)

Add the [`webpacker`](https://github.com/rails/webpacker) gem to your `Gemfile`:

``` ruby
gem "webpacker"
```

Bundle, and then run the webpacker installer:

``` bash
$ bundle install
$ rails webpacker:install
```

#### For a new rails app (DO THIS INSTEAD)

```
$ rails new FrankensteinApp --webpack

# however Nathan’s preference for this app is:
# $ rails new FrankensteinApp --webpack --database=postgresql --skip-coffee --skip-action-mailer --skip-test
```

> Note that you now have a new `app/javascripts/packs/application.js` file, specifically for bundling together and serving your javascript dependencies with webpack (using the asset pipeline!).

## React on Webpack on Rails

Add the [`react-rails`](https://github.com/reactjs/react-rails) gem to your `Gemfile`:

``` ruby
gem "react-rails"
```

Bundle, and then run the `react-rails` installer:

``` bash
$ bundle install
$ rails webpacker:install:react
$ rails g react:install
```

> If `rails g react:install` hangs forever, please open a new terminal, run `spring stop`, and try again.

> Note that you now have a new `components.js` file, and a `app/assets/javascripts/components/` folder for your react components.

#### Looking around...
Your updated project directory looks like this:

What's new?

```
.
├── app
│   ├── assets
│   │   ├── config
│   │   │   └── manifest.js
│   │   ├── images
│   │   ├── javascripts
│   │   │   ├── application.js
│   │   │   ├── components/...
│   │   │   └── components.js
│   │   └── stylesheets/...
│   ├── controllers/...
│   ├── helpers/...
│   ├── javascript
│   │   └── packs
│   │       └── application.js
│   ├── models/...
│   └── views/...
├── bin
│   ├── bundle
│   ├── rails
│   ├── webpack
│   ├── webpack-dev-server
│   └── yarn
├── config
│   ├── webpack/...
│   └── webpacker.yml
├── db/...
├── Gemfile
├── Gemfile.lock
├── config.ru
├── package.json
└── yarn.lock
```

## Plug the webpack script into your application layout
Our webpack-ed application javascript isn't yet available to the client.

In `app/views/layouts/application.html.erb` add the following line to your `<head>`:

```
<%= javascript_pack_tag 'application' %>
```

> This is different from your normal `./app/assets/javascripts/application.rb` files! It refers instead to the webpack manifest located at `./javascripts/packs/application.js`!

## Using react-rails generators / view helpers

**Create a component**

Just as Rails has a handy model generator (e.g. `rails g model Potato`) `react-rails` comes with a [component generator](https://github.com/reactjs/react-rails#component-generator) that works in much the same way:

``` bash
$ rails g react:component Person name:string dob:string --es6  # note: es6 is optional
```

> Keep in mind that our types now refer to javascript datatypes, NOT RUBY datatypes!!!

This will generate a new file called `app/assets/javascripts/components/person.es6.jsx`:

``` js
class Person extends React.Component {
  render () {
    return (
      <div>
        <div>Name: {this.props.name}</div>
        <div>Dob: {this.props.dob}</div>
      </div>
    );
  }
}

Person.propTypes = {
  name: React.PropTypes.string,
  dob: React.PropTypes.string
};
```

#### Create PersonsController and route

Generate a controller, with a `show` action:

```
$ rails g controller persons show
```

Add a homepage route, in `config/routes.rb`:

```ruby
root to: "persons#show"
```

#### Insert the component into the view

In `app/views/persons/show.html.erb` use the react-rails [`react_component` view helper](https://github.com/reactjs/react-rails#view-helper) with some stubbed data:

``` erb
<%= react_component "Person", { name: "Suzy", dob: "11/04/1985" }  %>
```

## Run your rails server *AND* your webpack server!!!
You will need to run your webpack server in addition to your normal rails server (just have 2 tabs open):

* `$ ./bin/webpack-dev-server` # on http://localhost:8080/`
* `$ rails server` # on http://localhost:3000/`


Now open your browser to `http://localhost:3000/`. You should see your component rendered.

Take a look at the raw source code in your browser (`View > Developer > View Source`).

Now compare that output to what happens when you use the [`prerender: true`](https://github.com/reactjs/react-rails#server-side-rendering) option:

``` erb
<%= react_component "Person", { name: "Suzy", dob: "11/04/1985" }, prerender: true  %>
```

> Prerendering opens the doors for what are called "isomorphic" applications.

**BONUS:** How would you tie in a Person model? Right now we're hard coding the data. How might we retrieve it from the databse?

## Resources

* [Rails 5.1 release notes](http://edgeguides.rubyonrails.org/5_1_release_notes.html)
* [Yarn package manager](https://yarnpkg.com/en/docs/usage) & [Commands Reference](https://yarnpkg.com/en/docs/usage)
* [`react-rails` gem](https://github.com/reactjs/react-rails)
* [`webpacker` gem](https://github.com/rails/webpacker)
* [AirBnB on the Isomorphic Application Pattern](https://medium.com/airbnb-engineering/isomorphic-javascript-the-future-of-web-apps-10882b7a2ebc)
