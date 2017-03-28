title: How to use a CakePHP API as the data backend for Ember in 30 minutes
date: 2017-03-13 18:00:00
tags:
  - cakephp
  - emberjs
  - ember
  - api
  - jsonapi
  - crud
categories:
  - cakephp
---

In this follow-up post to [How to make your CakePHP 3 API produce JSON API](/2017/03/how-to-make-your-cakephp-3-api-produce-jsonapi/)
we will show you how easy it is to use your CakePHP API as the backend for an [Ember](http://emberjs.com/)
application, allowing you to keep benefiting from the extremely powerful
[CakePHP ORM](https://book.cakephp.org/3.0/en/orm.html)
whilst also enjoying all the frontend-goodies provided by Ember.

The tutorial:

- provides step-by-step instructions
- ends with a full CRUD implementation
- adds background information to help you understand the underlying concepts

## CakePHP Users

Even though this tutorial does not touch a single piece of PHP code the learning
curve should be minimal since both frameworks share a lot of characteristics:

- Convention over configuration
- A stack of proven tools that "just work"
- Command line tools for baking/generating code
- Rapid Development; getting productive, very fast
- Highly maintainable, painless upgrades
- Suitable for both small and ambitous applications

## Before We Begin

This is part six of the CakePHP 3 REST API tutorial series:

1. [How to build a CakePHP 3 REST API in minutes](/2015/04/how-to-build-a-cakephp-3-rest-api-in-minutes/)
2. [How to use a CakePHP 3 REST API](http://cakebox:4000/2015/04/how-to-use-a-cakephp-3-rest-api/)
3. [How to prefix route a CakePHP 3 REST API](/2015/04/how-to-prefix-route-a-cakephp-3-rest-api/)
4. [How to add JWT Authentication to a CakePHP 3 REST API](/2015/04/how-to-add-jwt-authentication-to-a-cakephp-3-rest-api/)
5. [How to make your CakePHP 3 API produce JSON API](/2017/03/how-to-make-your-cakephp-3-api-produce-jsonapi/)
6. How to create an Ember application with (CakePHP) JSON API backend in 30 minutes

Before starting this tutorial **make absolutely sure**:

- that you have the `cake3api.app` API up-and-running
- that it is producing valid JSON API
- that it is allowing CORS requests

You may create the API by either:

- completing [the previous tutorial](2017/03/how-to-make-your-cakephp-3-api-produce-jsonapi/)
- starting fresh by using these
 [end-state application sources](https://github.com/bravo-kernel/application-examples/tree/master/blog-how-to-make-your-cakephp-3-api-produce-jsonapi),
composer installing and running the database migration.

## Assumptions

This tutorial assumes:

- the hostname serving your Ember application is [cakebox](https://github.com/alt3/cakebox) (replace with your own system name where applicable)
- the FQDN of your API is `cake3api.app` (replace with your own FQDN where applicable)
- Chrome with the [Ember Inspector add-on](https://chrome.google.com/webstore/detail/ember-inspector/bmdblncegkenkacieihfhpjfppoconhi) as your client browser

## 1. TLDR; just give me the Ember end-state source files

Novice users should skip this step.

```bash
git clone https://github.com/bravo-kernel/application-examples
cd application-examples
cd blog-how-to-use-a-cakephp-api-as-the-data-backend-for-ember-in-30-minutes
npm install
ember serve
```

Browse to `http://cakebox:4200`

## 2. Requirements

This tutorial requires the installation of
[nodejs](https://nodejs.org/en/),
[ember-cli](https://ember-cli.com/),
[phantomjs](http://phantomjs.org/),
and
[bower](https://bower.io/) on your server.

Since every system will have different requirements you should consider the following
instructions (taken from an installation on [cakebox](https://github.com/alt3/cakebox)
using Ubuntu 14.04) as nothing more than an example:

```bash
curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
sudo apt-get update
sudo apt-get install nodejs
npm install -g ember-cli
npm install -g phantomjs
npm install -g bower
```

## 3. Creating the Ember application

Ember comes with the `ember` CLI command which is similar to CakePHP's `cake` and used
to generate applications, routes, models, etc. We will generate a fresh Ember application
named `ember-frontend` by running:

```bash
cd /path/to/your/projects/directory
ember new ember-frontend
cd ember-frontend
```

<br />

{% asset_img 01-ember-cli-new-application.png 'Screenshot of ember-cli generating the Ember application' %}

## 4. Meeting Ember

Since this is basically all that is needed to create a new Ember application this might be a good moment
to take a quick look at the files and folders to get a quick first impression of the application structure.

To see your Ember application in action run `ember serve` and browse to `http://cakebox:4200`:

<br />

{% asset_img 02-ember-welcome-page.png 'Screenshot of Ember welcome page' %}

To continue this tutorial without the Tomster mascotte
open `templates/application.hbs` and remove the following line:

    {{welcome-page}}

## 5. Creating the Ember Data Adapter

Ember Data's (now default) [JSONAPIAdapter](http://emberjs.com/api/data/classes/DS.JSONAPIAdapter.html)
is the awesome/amazing glue responsible for automatically constructing all
JSON API calls required to communicate with your API backend. Generate it by running:

```bash
ember generate adapter application
```

Now open `app/adapters/application.js` and add the connection settings pointing to your API
so it looks like this:

```js
import DS from 'ember-data';

export default DS.JSONAPIAdapter.extend({
  host: "http://cake3api.app",
  namespace: "api"
});
```

*Please note that we MUST set the namespace here because we created a CakePHP prefixed route in [tutorial #3](http://www.bravo-kernel.com/2015/04/how-to-prefix-route-a-cakephp-3-rest-api/)*

## 6. Creating the Ember models

Ember Data must somehow be made aware of the models/resources available in our CakePHP API
(and their relationships) and amazingly enough all that is required for that to happen is
generating an "Ember model".


Since our API is serving cocktails we create a corresponding Ember model by running:

```bash
ember generate model cocktail
```

Now open `app/models/coctkail.js` and specify which attributes you want to be available in the frontend:

```js
import DS from 'ember-data';
import attr from 'ember-data/attr';

export default DS.Model.extend({
  name: attr(),
  description: attr(),
  created: attr(),
});
```

*Do not forget to add `ember-data/attr`*

## 7. Adding the index route

To start with Ember uses "routes" which are comparable with CakePHP controllers
and [Handlebars](http://handlebarsjs.com/) "templates" which are comparable with
CakePHP templates. To generate the route we will use to hook into our API's `index`
action we run:

```bash
ember generate route cocktails/index
```

This will:

- create `app/routes/cocktails/index.js`
- create `app/templates/cocktails/index.hbs`
- update `app/router.js` to include the newly generated route

Now open `app/routes/cocktails/index.js` and add our model so **all** cocktail data is
automatically fetched from our API when accessing this route/page.

```js
import Ember from 'ember';

export default Ember.Route.extend({
  model() {
    return this.store.findAll('cocktail');
  },
});
```

All that is needed now is updating the template so it will show the
fetched data so let's update `app/templates/cocktails/index.js` and make
it look like this:

```html
<h2>Ember Data JSONAPIAdapter - Index route</h2>

<table>
  {{#each model as |cocktail|}}
      <tr>
          <td>{{cocktail.id}}</td>
          <td>{{cocktail.name}}</td>
          <td>{{cocktail.description}}</td>
          <td>{{cocktail.created}}</td>
      </tr>
  {{/each}}
</table>
```

Done, seriously.

### Verify

Now that you have created your first API-driven page verify Ember is actually fetching from your API:

- open Chrome
- browse to your Ember application at `cakebox:4200/cocktails`
- make sure it displays the records as found in your CakePHP database:

<br />

{% asset_img 03-ember-showing-data-from-cakephp-api-index-action.png 'Ember showing data fetched from CakePHP API' %}

## 8. Using Ember Inspector

The Ember Inspector in Chrome will help you get a better understanding of the underlying
API magic so let's zoom in on that subject before continuing.

### Inspecting API Resources

The API resources used by your Ember frontend can be found by:

- opening chrome
- pressing F12
- selecting the `Network` tab
- browsing to `cakebox:4200/cocktails`

<br />

{% asset_img 04-ember-inspector-network.png 'Ember Inspector API network resource' %}

  - **upper arrow**: tells us that a network resource named `cocktails` was fetched
using jquery (executed by Ember Data)
  - **lower arrow**: tells us Ember Inspector is recognizing the page as valid Ember


### Inspecting API Resource Headers

To zoom in on the API resource and inspect the headers simply click `cocktails` which
should give you something like:

<br />

{% asset_img 05-ember-inspector-api-resource-zoom-headers.png 'Ember Inspector zoom in on API network resource headers' %}

- **upper arrow**: proves the resource is fetched from our CakePHP API
- **center arrow**: tells us the CakePHP API is responding with JSON API
- **lower arrow**: tells us Ember Data is automatically sending the JSON API Accept Header with all requests

### Inspecting the API response

You will be able to see the JSON API response produced by the API by clicking
the `Response` tab. This should look more than familiar since you have implemented
JSON API during the previous tutorial (pretty cool).

*FYI the `Preview` tab will show the exact same results but objectied for easy click-through.*


<br />

{% asset_img 06-ember-inspector-api-resource-response.png 'Ember Inspector show API response' %}

### Inspecting Ember Data records

The Ember Inspector is really useful when we want to see how our
API data is being used by Ember. To make this more clear:

- select the `Ember` tab
- select the `Data` container
- select the `cocktail` model type

You should notice two things:

- the attributes are corresponding to the Ember `cocktail` model we created earlier
- and are thus missing the `modified` attribute we excluded (even though our API is servicing it)

<br />

{% asset_img 07-ember-inspector-api-data-main.png 'Ember Inspector Ember Data main' %}

### Inspecting Ember Data record details

Every record contains very detailed information which is revealed when clicking a record.

*Relationship information (`belongsTo`, `hasMany`) will appear in the `Flags` section when applicable.*

<br />

{% asset_img 08-ember-inspector-api-record-details.png 'Ember Inspector Ember Data record details' %}

## 9. Adding the delete action

Ember uses [template actions](https://guides.emberjs.com/v1.13.0/templates/actions/)
to let users interact with data so let's add one
that will automatically construct and send the correct `DELETE` request
to our API.

Moment, since all template actions require a corresponding (named) event in the routes file
first open `app/routes/cocktails/index.js` and update it to look like this:

```js
import Ember from 'ember';

export default Ember.Route.extend({
  model() {
    return this.store.findAll('cocktail');
  },

  actions: {
    deleteCocktail(cocktail) {
      let confirmation = confirm('Are you sure?');

      if (confirmation) {
        cocktail.destroyRecord();
      }
    }
  }
});
```

Now that the `deleteCocktail` event/action is in place all that is left to do is updating
the index template with a button so users can trigger the action. To do so, open `templates/cocktails/index.hbs`
and update it to look like this:

```
<h3>Ember showing data fetched from CakePHP API's index action</h3>
<table>
  {{#each model as |cocktail|}}
      <tr>
          <td>{{cocktail.id}}</td>
          <td>{{cocktail.name}}</td>
          <td>{{cocktail.description}}</td>
          <td>{{cocktail.created}}</td>
          <td><button class="button" {{action 'deleteCocktail' cocktail}}>Delete</button></td>
      </tr>
  {{/each}}
</table>
```

### Verify

Verify you are able to delete records in your API by using the Ember page by:

- opening Chrome
- pressing F12
- selecting the `Network` tab
- browsing to your Ember application at `cakebox:4200/cocktails`
- making sure it now displays `Delete` buttons
- pressing one of the `Delete` buttons (and confirming the delete)

<br />

{% asset_img 09-delete-action-page.png 'Ember delete action page' %}

If things went well:

- Ember automagically made two requests to your API (see the Chrome `Network` tab):
  - one for the CORS `OPTIONS`
  - one for the actual `DELETE`
- clicking those requests should now produce information that makes sense to you
- the record was deleted from your CakePHP database
- refreshing the page should no longer show the deleted record
- you should by now be quite impressed with how easy that actually was

<br />

{% asset_img 10-delete-action-two-requests.png 'Ember delete action two requests' %}

## 10. Adding the add action

Without going into details we will create an
[Ember nested route](https://guides.emberjs.com/v2.11.0/tutorial/subroutes/#toc-toggle)
to create a dedicated page for our `add` action by running:

```bash
ember generate route cocktails/add
```

This will:

- create `app/routes/cocktails/add.js`
- create `app/templates/cocktails/add.hbs`
- add the nested route to `router/router.js`


Similar to the `delete` action we will first create a named event
so update `app/routes/cocktails/add.js` to look like this:

```js
import Ember from 'ember';

export default Ember.Route.extend({
  model() {
    return this.store.createRecord('cocktail');
  },

  actions: {
    addCocktail(cocktail) {
      let confirmation = confirm('Are you sure?');

      if (confirmation) {
        cocktail.save();
      }
    }
  }

});
```

Open `app/templates/cocktails/add.hbs` and add this basic form
the user will use to trigger the `addCocktail` action:

```html
<h3>New Cocktail:</h3>
<form>
  <label>Name: {{input value=model.name}} </label>
  <label>Description: {{input value=model.description}} </label>
  <button class="button" {{action 'addCocktail' model}}>Add</button>
</form>
```

### Verify

Verify you can add a new record to your CakePHP database using your
Ember frontend by:

- browsing to `cakebox:4200/cocktails/add`
- making sure the input fields are there
- filling the form fields `name` and `description`
- pressing the `Add` button

If things went well:

- Ember automagically made two requests to your API (see the Chrome `Network` tab):
  - one for the CORS `OPTIONS`
  - one for the actual `POST` request
- you should be able to see the new record in your CakePHP database

<br />

{% asset_img 11-add-action-page.png 'Screenshot of Ember add action page' %}

## 11. Adding the view/edit actions

Almost there.

```bash
ember generate route cocktails/edit
```

Update `app/routes/cocktails/edit.js` so it looks like this:

```js
import Ember from 'ember';

export default Ember.Route.extend({
  model(params) {
    return this.get('store').findRecord('cocktail', params.id);
  },

  actions: {
    saveCocktail(country) {
      let confirmation = confirm('Are you sure?');

      if (confirmation) {
        country.save();
      }
    }
  }
});
```

To handle the `id` parameter open `app/router.js` and replace
`this.route('edit');` with:

        this.route('edit', { path: "/edit/:id" });


Update `app/templates/cocktails/edit.hbs` so it looks like this:

```html
<h3>Edit existing Cocktail (id = {{model.id}})</h3>
<form>
    <div>{{input value=model.name}}</div>
    <div>{{input value=model.description}}</div>
</form>

<button class="button alert small" {{action 'saveCocktail' model}}>Save</button>
```

All done, browse to `http://cakebox:4200/cocktails/edit/3` and verify that you can
view, edit and save the record.

<br />

{% asset_img 12-view-edit-action-page.png 'Screenshot of Ember view/edit action page' %}

## Bonus: Relational Data

For simplicity's sake we did not touch the subject in this tutorial but... Ember Data will
also (automatically) detect and handle your API's `belongsTo` and `hasMany` relationships
as can seen in the following screenshot:

<br />

{% asset_img 13-relationships.png 'Screenshot of Ember data relationships' %}

Please consult the Ember documentation for more information but enabling the
above relationships was as easy as adding them to the
Ember model like this:

```js
export default DS.Model.extend({
  name: attr(),
  code: attr(),
  native: attr(),
  currency: belongsTo('currency'),
  cultures: hasMany('culture')
});
```

## Bonus: Validation Errors

Because your CakePHP API is producing JSON API compatible (validation)
errors... those are handled by Ember Data automatically as well, making
them instantly availble in your frontend as can be seen in the
screenshot below where:

- your API is returning the 422 validation error along with validation error messages
- Ember Data picks up the validation messages and makes them avaible for display on the page

<br />

{% asset_img 14-validation-errors.png 'Screenshot of Ember data validation errors' %}

## Getting help

Both CakePHP and Ember have very friendly online communities that are more than willing to help
you out if you encounter any problems so make sure to join:

- the [CakePHP Slack group](https://www.google.nl/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&cad=rja&uact=8&ved=0ahUKEwivsN6Kq9TSAhVDExoKHTFXDuMQFggaMAA&url=https%3A%2F%2Fcakesf.herokuapp.com%2F&usg=AFQjCNF2de5Mi16W0okJJ9lsgy9JrDmiNQ&bvm=bv.149397726,d.d2s) or their
[#cakephp IRC channel](https://kiwiirc.com/client/irc.freenode.net#cakephp)
- the [Ember Slack group](https://embercommunity.slack.com/) or their [#emberjs IRC channel](https://kiwiirc.com/client/irc.freenode.net#emberjs)

## Additional reading

- [Ember.js - Guides and Tutorials](https://guides.emberjs.com/)
- [Why I prefer Ember.js over Angular & React.js](http://voidcanvas.com/prefer-ember-js-angular-react-js/)

*Eternal hat-tip to [Phally](https://github.com/Phally) for inspiring (and actually prototyping) this many years ago.*
