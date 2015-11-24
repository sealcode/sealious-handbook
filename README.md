## Field-Types

#### I. What are field-types?
Each "field" in a ResourceType must have a field-type assigned. Field-types describe which values can and which cannot be assigned to a field. Field-type's behaviour can be adjusted using field-type parameters.

A field-type can accept or reject a value, with appropriate error message.

It's a field-type's responsibility to describe how to store it's values in a datatore.

Some field-types that come bundled with Sealious:

 * `text` - for storing strings of chars.
 * `email` - like text, but accepts only valid email addresses as values
 * `int` - for storing integer numbers
 * `float` - for storing real numbers
 * `file` - for storing managed user-submitted files
 * `reference` - for storing a reference to a resource. This field-type can be adjusted to only accept references to certain resource types.

 File `lib/base-chips/_base-chips.js` defines the order field-type initilizing.

#### II. How to use field-types?
The example below shows a simple Sealious app:
```js
1   var Sealious = require("sealious");
2  
3   Sealious.init();
4 
5   new Sealious.ChipTypes.ResourceType({
6      name: "person",
7      fields: [
8          {name: "name", type: "text", params: {max_length: 25}, required: true},
9          {name: "age", type: "int", required: true},
10         {name: "hair-color", type: "color"}
11     ]
12  });
13
14  Sealious.ChipManager.get_chip("channel", "rest").set_url_base("/api/v1");
15
16  Sealious.start();
```

As you can see, there are three fields declared:
    
 * `name`, which is of `text` type,
 * `age`, which is of `int` type,
 * `hair-color`, which is of `color` type.

Each field has a type assigned which describes how the field behaves and what value it contains. 
For example, `field-type.color` accepts values like `"black"`, `"#000000"`, 
or `{r: 0, g: 0, b: 0}`, but rejects values such as `"this is my color"`.

Also notice, that `name` field has defined an object called `params`, which provides additional information about the field-type. In this case, `name` can have no more than `120` characters.
Every field can have `params` object to specify what values can be accepted and what rejected.


**Q**: *But how can I actually use those field-types?*

**A**: Good question. Let's take a look at line **14**. 

This line tells us, that you can send HTTP requests on url `/api/v1`. 
This means that you can send, for example, HTTP POST request on (deafult) URL `http://localhost/api/v1/person` with this body:
![HTTP POST request](http_post.png)

This will add a new user named **Maurice**, who is **21** years old and has **blue** hair color to the database.

#### III. Field-types in Sealious
Sealious comes with 11 pre-defined field-types:

1. Boolean
2. Color
3. Date
4. Datetime
5. Email
6. File
7. Float
8. Hashed-text
9. Int
10. Reference
11. Text

#### IV. Creating a new field-type

#### V. Common questions and errors

**Q**: I created a new field-type, I want to use it in my app, but when I create a new field with my field-type and start the app, I get this error:
    ```js
    Error: In declaration of resource type 'person': unknown field type 'my-new-field-type' in field 'name'.
    
    ```

**A**: There are several causes that may throw this error (**not** including typos):

1. Did you remember to add your new field-type reference to `/lib/base-chips/_base-chips.js`? It should look like this:
    ```js
    require("./field_type.color.js");
    require("./field_type.file.js");
    require("./your-new-field-type.js")
    ```

2. Did you remember to include `name` property in your field-type declaration?
    ```js
    new Sealious.ChipTypes.FieldType({
        name: "my-new-field-type",
        //rest of the declaration
    });
    ```


---


**Q**: I used my new field-type in my app declaration, everything is okay when the app starts, but when I try to use HTTP POST request, nothing happens. No error, no response, nothing.

**A**: Make sure that in your field-type declaration, the `is_proper_value` method uses `accept()` argument.
    ```js
    is_proper_value: function(accept, reject, context, params, number){
        var test = parseFloat(number);
        if (test === null || test === NaN || isNaN(number) === true) {
            reject("Value `" + number + "` is not a float number format.");
        } else {
            accept();
        }
    }
    ```

---


**Q**: I try to create a new field-type that inherits (extends) from an already-existing field-type. I added the reference to `/lib/base-chips/_base-chips.js`, but all I get is this error:
    ```
    Error: ChipManager was asked to return a chip of type `field_type` and name `<already-existing-field-type>`, but it was not found
    ```

**A**: Okay, so I assume you are sure, that this already-existing field-type indeed exists and is referenced in `/lib/base-chips/_base-chips.js`. 

File `/lib/base-chips/_base-chips.js` contains a list of currently used field-types. Those field-types are loaded one after another, sequentially. This error occurs, when you want to load your field-type **before** the already existing field-type you want to inherit from.

To solve this problem, make sure that Sealious first loads that existing field-type, and then yours:
```js
require("./already-existing-field-type.js")
require("./your-field-type.js");
```

