## Simple Sealious app
In this section we will show you how to create a fully functional and working back-end application, with REST routes and database handling set, in just **28** lines:

```js
1   var Sealious = require("sealious"); 
2
3   Sealious.init();
4
5   require("./field-type.animal.js");
6
7   new Sealious.ChipTypes.ResourceType({
8       name: "owner",
9       fields: [
10          {name: "first-name", type: "text", required: true},
11          {name: "last-name", type: "text", required: true},
12          {name: "address", type: "text", required: true},
13          {name: "phone-number", type: "int", required: true}
14          {name: "email", type: "email"}
15      ]
16  });
17
18  new Sealious.ChipTypes.ResourceType({
19      name: "pet",
20      fields: [
21          {name: "species", type: "animald", required: true},
22          {name: "name", type: "text", required: true},
23          {name: "age", type: "int", required: true},
24          {name: "diagnosis", type: "text", params: {max_length: 200}, required: true}
25     ]
26  });
27
28  Sealious.start();
```

We'll call this project **Veterinarian Clinic**. Basically it's main task is to store information about pets and their owners.


### Declaring our first ResourceType

ResourceType is the core concept of Sealious. Without it, Sealious has nothing to work on.

Let's create a new resource named `owner`.

In the real world, a pet must have a name and some age. Here's how we'll represent it in Sealious:

```js
1   var Sealious = require("sealious"); 
2
3   Sealious.init();
4
5   new Sealious.ChipTypes.ResourceType({
6       name: "owner",
7       fields: [
8           {name: "first-name", type: "text", required: true},
9           {name: "last-name", type: "text", required: true},
10          {name: "address", type: "text", required: true},
11          {name: "phone-number", type: "int", required: true}
12          {name: "email", type: "email"}
13      ]
14  });
15
16  Sealious.start();
```

This app consists of four parts:

 * `var Sealious = require("sealious");` - a reference to Sealious,
 * `Sealious.init();` - loads Sealious components,
 * `new Sealious.ChipTypes.ResourceType({})` - defines a new resource-type that will be our data model,
 * `Sealious.start();` - prepares REST routes, sets the database, loads dependencies - gets the whole thing started.

This is all we need to start a new Sealious app.

From now on we can communicate with the server through REST routes (the default Sealious channel):

 * `GET` on URL `api/v1/owner` returns all `owner` resources,
 * `GET` on URL `api/v1/owner/{owner_id}` returns a specific `owner` resource,
 * `POST` on URL `api/v1/owner` with body `{name: <text>, age: <int>}` creates a new `owner` resource with given body,
 * `PUT` and `PATCH` on URL `api/v1/owner/{owner_id}` with body `{name: <text>, age: <int>}` modifies the `owner` resource,
 * `DELETE` on URL `api/v1/owner/{owner_id}` deletes a specific `owner` resource.


---

**Q**: *How can I communicate with the server if I don't want to use REST?*

**A**: We give the developers channel REST by default. If you want to use other channel you are free to write one yourself. See X for more information.

---

**Q**: *I have no idea what's happening in this ResourceType thing...*

**A**: Let's walk through it step by step:

1. `new Sealious.ChipTypes.ResourceType({` - this line creates a new `ResourceType` object, that will enable you to work on your resources.
2. `name: "owner",` - this is the name of our resource, that we can work with.
3. `fields: [` - we define `fields` array that holds information about what type of data our resource can store.
4. `{name: "first-name", type: "text", required: true},` - this is the field used by our resource. The name of the field is `name`, its type is `text` and it must be delcared (`required: true`).
5. `{name: "last-name", type: "int", required: true}` - this is the another field used by our resource. The name of the fields is `age`, its type is `int` and it must be declared (`required: true`).
6. `{name: "email", type: "email"}` - this defines the field named `email` with type `email` - it only accepts strings that look like and email. Note that `required: true` is absent - that means, that you don't have to include this field and the request will still be parsed correctly.

And that's it.

If you want to know more about `ResourceType`, see X.

If you want to know more about `FieldType`, see X.

### Adding more resources

You can add as many resources as you need. In our example, we'll add `pet` resource.

```js
1   var Sealious = require("sealious"); 
2
3   Sealious.init();
4
5   new Sealious.ChipTypes.ResourceType({
6       name: "owner",
7       fields: [
8           {name: "first-name", type: "text", required: true},
9           {name: "last-name", type: "text", required: true},
10          {name: "address", type: "text", required: true},
11          {name: "phone-number", type: "int", required: true}
12          {name: "email", type: "email"}
13      ]
14  });
15
16  new Sealious.ChipTypes.ResourceType({
17      name: "pet",
18      fields: [
19          {name: "species", type: "text", required: true},
20          {name: "name", type: "text", required: true},
21          {name: "age", type: "int", required: true},
22          {name: "diagnosis", type: "text", params: {max_length: 200}, required: true}
22      ]
23  });
24
25  Sealious.start();
```

As you can see, the `pet` resource has *four* fields:

1. `spieces`,
2. `name`,
3. `age`,
4. `diagnosis`.

The new thing here is `params: {max_length: 200}` in field `diagnosis`. This parameter says, that only strings with length up to 200 characters.

And that's it. We've finished our **Veterinarian Clinic**!

---

**Q**: *But wait, the first code block you showed us had line `require("./field-type.animal.js");`. What's with this?*

**A**: Nice eye. Here's the explanation:

Sealious let's you define your own field-types, that will validate, encode or decode (and many more - see API reference) the values that's in the field.

For our example, we've defined a new field-type, that checks if given animal species is acceptable by our clinic:

```js
1   var Sealious = require("sealious");
2
3   new Sealious.ChipTypes.FieldType({
4       name: "animal",
5       get_description: function(context, params){
6           return "Only accepts dogs, cats and parrots.";
7       },
8       is_proper_value: function(accept, reject, context, params, new_value){
9           var acceptable_animals = ['dog', 'cat', 'parrot'];
10          var result = acceptable_animals.find( x => x === new_value);
11
12          result ? accept() : reject("This species of animal is not accepted");
13      }
14  });
```

If you want to know more about `FieldType`, see X.

Now if we include our newly created field-type, we can check what's the spieces of the pet we want to add to our resources.


Final result:
```js
1   var Sealious = require("sealious"); 
2
3   Sealious.init();
4
5   require("./field-type.animal.js");
6
7   new Sealious.ChipTypes.ResourceType({
8       name: "owner",
9       fields: [
10          {name: "first-name", type: "text", required: true},
11          {name: "last-name", type: "text", required: true},
12          {name: "address", type: "text", required: true},
13          {name: "phone-number", type: "int", required: true}
14          {name: "email", type: "email"}
15      ]
16  });
17
18  new Sealious.ChipTypes.ResourceType({
19      name: "pet",
20      fields: [
21          {name: "species", type: "animald", required: true},
22          {name: "name", type: "text", required: true},
23          {name: "age", type: "int", required: true},
24          {name: "diagnosis", type: "text", params: {max_length: 200}, required: true}
25     ]
26  });
27
28  Sealious.start();
```
