## Creating Records

You can create records by calling the
[`createRecord()`](http://emberjs.com/api/data/classes/DS.Store.html#method_createRecord)
method on the store.

```js
store.createRecord('post', {
  title: 'Rails is Omakase',
  body: 'Lorem ipsum'
});
```

The store object is available in controllers and routes using `this.get('store')`.

## Updating Records

Making changes to Ember Data records is as simple as setting the attribute you
want to change:

```js
this.get('store').findRecord('person', 1).then(function(tyrion) {
  // ...after the record has loaded
  tyrion.set('firstName', "Yollo");
});
```

All of the Ember.js conveniences are available for
modifying attributes. For example, you can use `Ember.Object`'s
[`incrementProperty`](http://emberjs.com/api/classes/Ember.Object.html#method_incrementProperty) helper:

```js
person.incrementProperty('age'); // Happy birthday!
```

## Persisting Records

Records in Ember Data are persisted on a per-instance basis.
Call [`save()`](http://emberjs.com/api/data/classes/DS.Model.html#method_save)
on any instance of `DS.Model` and it will make a network request.

Ember Data takes care of tracking the state of each record for
you. This allows Ember Data to treat newly created records differently
from existing records when saving.

By default, Ember Data will `POST` newly created records to their type url.

```javascript
var post = store.createRecord('post', {
  title: 'Rails is Omakase',
  body: 'Lorem ipsum'
});

post.save(); // => POST to '/posts'
```

Records that already exist on the backend are updated using the HTTP `PATCH` verb.

```javascript
store.findRecord('post', 1).then(function(post) {
  post.get('title'); // => "Rails is Omakase"

  post.set('title', 'A new post');

  post.save(); // => PATCH to '/posts/1'
});
```

You can tell if a record has outstanding changes that have not yet been
saved by checking its
[`hasDirtyAttributes`](http://emberjs.com/api/data/classes/DS.Model.html#property_hasDirtyAttributes)
property. You can also see what parts of
the record were changed and what the original value was using the
[`changedAttributes()`](http://emberjs.com/api/data/classes/DS.Model.html#method_changedAttributes)
method. `changedAttributes` returns an object, whose keys are the changed
properties and values are an array of values `[oldValue, newValue]`.

```js
person.get('isAdmin');            //=> false
person.get('hasDirtyAttributes'); //=> false
person.set('isAdmin', true);
person.get('hasDirtyAttributes'); //=> true
person.changedAttributes();       //=> { isAdmin: [false, true] }
```

At this point, you can either persist your changes via `save()` or you can roll
back your changes. Calling
[`rollbackAttributes()`](http://emberjs.com/api/data/classes/DS.Model.html#method_rollbackAttributes)
for a saved record reverts all the `changedAttributes` to their original value.
If the record `isNew` it will be removed from the store.

```js
person.get('hasDirtyAttributes'); //=> true
person.changedAttributes();       //=> { isAdmin: [false, true] }

person.rollbackAttributes();

person.get('hasDirtyAttributes'); //=> false
person.get('isAdmin');            //=> false
person.changedAttributes();       //=> {}
```

## Handling Validation Errors

If the backend server returns validation errors after trying to save, they will
be available on the `errors` property of your model. Here's how you might display
the errors from saving a blog post in your template:

```hbs
{{#each post.errors.title as |error|}}
  <div class="error">{{error.message}}</div>
{{/each}}
{{#each post.errors.body as |error|}}
  <div class="error">{{error.message}}</div>
{{/each}}
```

## Promises

[`save()`](http://emberjs.com/api/data/classes/DS.Model.html#method_save) returns
a promise, which makes easy to asynchronously handle success and failure 
scenarios.  Here's a common pattern:

```javascript
var post = store.createRecord('post', {
  title: 'Rails is Omakase',
  body: 'Lorem ipsum'
});

var self = this;

function transitionToPost(post) {
  self.transitionToRoute('posts.show', post);
}

function failure(reason) {
  // handle the error
}

post.save().then(transitionToPost).catch(failure);

// => POST to '/posts'
// => transitioning to posts.show route
```

## Deleting Records

Deleting records is as straightforward as creating records. Call [`deleteRecord()`](http://emberjs.com/api/data/classes/DS.Model.html#method_deleteRecord)
on any instance of `DS.Model`. This flags the record as `isDeleted`. The 
deletion can then be persisted using `save()`.  Alternatively, you can use 
the `destroyRecord` method to delete and persist at the same time.

```js
store.findRecord('post', 1).then(function(post) {
  post.deleteRecord();
  post.get('isDeleted'); // => true
  post.save(); // => DELETE to /posts/1
});

// OR
store.findRecord('post', 2).then(function(post) {
  post.destroyRecord(); // => DELETE to /posts/2
});
```
