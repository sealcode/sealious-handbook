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

Several things to note:

1. As you can see, there are *three* fields declared:
    * `name`, which is of `text` type,
    * `age`, which is of `int` type,
    * `hair-color`, which is of `color` type.

2. Each field has a type assigned which describes how the field behaves and what value it contains. 
For example, `field-type.color` accepts values like:
    * `"black"`, 
    * `"#000000"`, 
    * `{r: 0, g: 0, b: 0}`,
but rejects values such as `"this is my color"`.

3. Field `name` has defined an object called `params`, which provides additional information about the field-type. 
In this case, `name` can have no more than `120` characters.
Every field can have `params` object to specify what values can be accepted and what rejected.

---

**Q**: *But how can I actually use those field-types?*

**A**: Good question. Let's take a look at line **14**:

```js
14  Sealious.ChipManager.get_chip("channel", "rest").set_url_base("/api/v1");
```

This line tells us, that you can send HTTP requests on url `/api/v1`. 
This means that you can send, for example, HTTP POST request on (deafult) URL `http://localhost/api/v1/person` with this body:
![HTTP POST request](http_post.png)

Sending this request will result in adding a new user named **Maurice**, who is **21** years old and has **blue** hair color to the database.

---

#### III. Field-types in Sealious
Sealious comes with *eleven* pre-defined field-types:

1. Boolean 
    * boolean value: `true` or `false`,
    * can be a string: `"true"` or `"false"`, 
    * as well as numeric: `1` or `0`. 
2. Color
    * `"black"`, 
    * `"#000000"`, 
    * `{r: 0, g: 0, b: 0}`. 
3. Date
    * date standard ISO 8601 `YYYY-MM-DD`,
    * rejects other date formats.
4. Datetime
    * timestamp - amount of miliseconds since epoch.
5. Email
    * email address, like something@something.sth.
6. File
7. Float
    * a float number.
8. Hashed-text
    * hash with chosen algorithm (default `md5`),
    * designed to hash passwords:
        * can specify 1 or more required numbers,
        * can specify 1 or more required capital letters,
        * can specify 1 or more required numbers and capital letters.
9. Int
    * an integer number.
10. Reference
11. Text
    * can specify maximum string length,
    * can specify minimum string length.

#### IV. Creating a new field-type
In this section we will show you step by step how to create your own field-type.
We will use `field-type.color` for this example.

**Ready?**

1. Create your field-type file in `lib/base-chips/`.
2. Add reference to `lib/base-chips/_base-chips.js`:
    ```js
    // some requires...
    require("./field_type.color.js");
    require("./field_type.file.js");
    require("./your-new-field-type.js") // add this
    // some requires...
    ```

3. Write the skeleton of your new field-type.
```js
1   var Sealious = require("sealious");
2
3   new Sealious.ChipTypes.FieldType({
4        
5   });
```

4. The most basic form of field-type consists of `name` property and `is_proper_value` method. Let's add them!
```js
1   var Sealious = require("sealious");
2
3   new Sealious.ChipTypes.FieldType({
4       name: "color",
5       is_proper_value: function(accept, reject, context, params, new_value){
6
7       }
8   });
```
*Note*: `is_proper_value` method checks if `new_value` correct. If not, it rejects the request.

5. Since we want to create a field-type that will accept colors, we have to implement `is_proper_value` method properly, so that it will accepts values like `"black"` or `"#123FA5"` and reject values like `"silly sealy"`.
In this example we will use a module from `npm` called `color`.
```js
1   var Sealious = require("sealious");
2   var Color = require("color");
3
4   new Sealious.ChipTypes.FieldType({
5       name: "color",
6       is_proper_value: function(accept, reject, context, params, new_value){
7           try {
8               if (typeof (new_value) === "string")
9                   Color(new_value.toLowerCase());
10              else
11                  Color(new_value);
12          } catch (e){
13              reject("Value `" + new_value + "` could not be parsed as a color.");
14          }
15          accept();
16      },
17  });
```
And now, `is_proper_value` will accept those colors, that can be parsed correctly by `color` module. If the parsing cannot be done, the method will reject the `new_value` argument.

6. Voila! We have created a new field-type that accepts colors! Is it all?
Well, we can add `encode` method, which transforms (or encodes) the value that was given to `is_proper_value` to what we want. 

Let's say we want to store some colors in the database. This may be troublesome, because some colors can be named explicitly `black`, some can be rgb objects like `{r: 10, g: 2, b: 200}` and some can have hex values like `"#115DFA"`. This could be a mess!

So the safer approach would be parsing the colors to one, standarized form. This is what `encode` ensures.

```js
1   var Sealious = require("sealious");
2   var Color = require("color");
3
4   new Sealious.ChipTypes.FieldType({
5       name: "color",
6       is_proper_value: function(accept, reject, context, params, new_value){
7           try {
8               if (typeof (new_value) === "string")
9                   Color(new_value.toLowerCase());
10              else
11                  Color(new_value);
12          } catch (e){
13              reject("Value `" + new_value + "` could not be parsed as a color.");
14          }
15          accept();
16      },
18      encode: function(context, params, value_in_code){
19          var color = Color(value_in_code);
20          return color.hexString();
21      }
22  });
```
And that's it. Now you can use `color` as a field-type in your app field delcaration.


#### V. Common questions and errors

---

**Q**: *I created a new field-type, I want to use it in my app, but when I create a new field with my field-type and start the app, I get this error:*
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


**Q**: *I used my new field-type in my app declaration, everything is okay when the app starts, but when I try to use HTTP POST request, nothing happens. No error, no response, nothing.*

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


**Q**: *I try to create a new field-type that inherits (extends) from an already-existing field-type. I added the reference to `/lib/base-chips/_base-chips.js`, but all I get is this error:*
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
