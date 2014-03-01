
ngbs-forms
==========

> Generate forms using Angular for validations and Bootstrap for styles from short and concise descriptions of the fields.

 * [Install](#install)
 * [Getting started](#getting-started)
 * [API](#API)
 * [Form files](#form-files)
 * [Form descriptors reference](#form-descriptors-reference)
 * [Fields and their descriptors reference](#fields-reference)
     * [Input field](docs/fields/input.md)
     * [Textarea field](docs/fields/textarea.md)
     * [Select field](docs/fields/select.md)
     * [Submit field](docs/fields/submit.md)
     * [Checkbox field](docs/fields/checkbox.md)
     * [Radio field](docs/fields/radio.md)
     * [Static field](docs/fields/static.md)
     * [StaticNoWrap field](docs/fields/staticNoWrap.md)
 * [Validators reference](docs/validators.md)


## <a name="install"></a> Install

```shell
npm install --save-dev ngbs-forms
```

There are plugins for [Grunt](https://github.com/ernestoalejo/grunt-ngbs-forms)
and [Gulp](https://github.com/ernestoalejo/gulp-ngbs-forms).


## <a name="getting-started"></a> Getting started

Create a `example.frm` file containing this:

```
form {
  fields {
    input foo {}
  }
}
```

Then you can compile it from Javascript:

```js
var forms = require('ngbs-forms'),
    fs = require('fs');

var contents = fs.readFileSync('example.frm');
var source = forms.parse(contents.toString());
var generated = forms.generate(source);
fs.writeFileSync('example.html');
```

Then include somehow the partial in your app (using ng-include or ng-view for example)
and use this javascript to manage the form:

```js
.controller('MyController', function($scope) {
  // Initial data of the form, you can provide default values if you want
  $scope.data = {};

  // Called when the form is submitted, save the data and do whatever you want
  $scope.submit = function() {
    MyEntity.save($scope.data).then(...);
  };
});
```


## <a name="API"></a> API

This library exposes two functions.

**`parse(strval)`**: Parse the string (read from a file for example) and return a
descriptor object containing the full structure of the file.

**`generate(descriptor)`**: Takes descriptor object generated by `parse`. Returns
a string with the generated HTML code.


## <a name="form-files"></a> Form files

Form files are written in a custom and concise language.

 * [Forms](#forms)
 * [Normal fields](#normal-fields)
 * [Submit fields](#submit-fields)
 * [Static fields](#static-fields)
 * [Validators](#validators)
 * [Comments](#comments)


### <a name="forms"></a> Forms

The basic structure of a file describes a form.

```
form {
  fields {
    [... my fields here ...]
  }
}
```

You can specify descriptors for the form, changing the name for example:

```
form {
  name = 'myformname'

  fields {
    [... my fields here ...]
  }
}
```


### <a name="normal-fields"></a> Normal fields

Most of the fields (like inputs, selects, checkboxes, etc.) receive a list of
descriptors and the validators for that field. Also they need an internal name.

For example, a simple input with the label and the placeholder setted. It will
save the data to "$scope.data.foo".

```
form {
  fields {
    input foo {
      label = 'My field label'
      placeholder = 'My input placeholder'
    }
  }
}
```

Some descriptors can be a list of (key, value) pairs. For example you can
pass additional attributes to the input tag using `attrs`.

```
form {
  fields {
    input foo {
      label = 'My label'

      attrs {
        style = 'width: 150px;'
        my-custom-attr = 'my-custom-value'
      }
    }
  }
}
```


### <a name="submit-fields"></a> Submit fields

Forms typically need a submit field. They don't need a name and have descriptors
like the normal ones. For example:

```
form {
  fields {
    input foo {}
    input bar {}

    submit {
      label = 'Send button'
    }
  }
}
```


### <a name="static-fields"></a> Static fields


There are two special type of fields that wrap static HTML and don't need any
descriptor. They're `static` and `staticNoWrap`. The first one will be wrapped
as if it would be a field (so you can style it pretty much like if it was one).
The second will be raw HTML added to the form that allows you to make sections,
wrap some fields in a frame, etc.

Some examples:

```
form {
  fields {
    input foo {}
    static {
      <p class="helper-block">Some static text added to the form</p>
    }

    staticNoWrap {
      <div class="panel panel-default">
        <div class="panel-body">
    }

    input bar {}
    input baz {}

    staticNoWrap {
        </div>
      </div>
    }
  }
}
```


### <a name="validators"></a> Validators

To add validators to the fields specify them in a `validators` section.
They can take arguments if needed. The error message will be displayed after the
first submit try for fields that don't pass the validation.

```
form {
  fields {
    input foo {
      label = 'My label'

      validators {
        required = 'Foo is required'
        minlength(3) = 'Foo should have at least 3 characters'
        regexp('/^[a-c]$/') = 'Foo should be composed of "a", "b" and "c" only.'
      }
    }
  }
}
```


### <a name="comments"></a> Comments

Finally, this wouldn't be a good language if things can't be commented out quickly.
You can make comments using the `/*` and `*/` syntax.

```
form {
  fields {
    input foo {} /* comment, this will get ignored */
    /*
      it can span several lines if you need it
     */
    select bar {}
  }
}
```


## <a name="form-descriptors-reference"></a> Form descriptors reference

 * [name](#form-name)
 * [objName](#form-objName)
 * [trySubmit](#form-trySubmit)
 * [submit](#form-submit)
 * [noFieldset](#form-noFieldset)


### <a name="form-name"></a> name
*Default*: `f[counter]`
*Type*: `string`

The name of the form. Things to take into account:

  * Changing this will the name of the variable Angular will scope to access
    this form ($scope.newname).
  * All the fields IDs are prefixed with the name of the form, so it should be as
    shorter as possible.
  * If not specified the library maintains a global counter to generate `f0`, `f1`
    and so on.


### <a name="form-objName"></a> objName
*Default*: `data`
*Type*: `string`

The name of the object that will store the form data. You can later access the
sent data using `$scope.data` or whatever objName you assign to the form.


### <a name="form-trySubmit"></a> trySubmit
*Default*: (empty)
*Type*: `string`

The name of a scoped function that will be called when the user tries to submit
the form but it hasn't be successful (it has validation errors). If empty no
function will be called.

For example, if you want to show an alert when the user tries to submit the form
(not a practical idea, but anyway) you'll have to specify `trySubmit`:

```
form {
  trySubmit = 'alertTheUser'

  fields {
    [... my fields here ...]
  }
}
```

Then you'll have to scope that function in the Angular controller:

```
.controller('MyController', function($scope) {
  $scope.data = {};

  // Called each time the user tries to send the form
  $scope.alertTheUser = function() {
    alert('Sending form...');
  };

  // Called when all the validations are successful
  $scope.submit = function() {
    // Whatever to send the form
  };
});
```


### <a name="form-submit"></a> submit
*Default*: `submit`
*Type*: `string`

The name of a scoped function that will be called when the user submits
the form.


### <a name="form-noFieldset"></a> noFieldset
*Default*: `false`
*Type*: `boolean`

If it's true the form won't be wrapped in a `fieldset` tag. Useful when you have
to specify special styles to make the form inline with other elements
(navbar for example)


## <a name="fields-reference"></a> Fields and their descriptors reference

 * [Input field](docs/fields/input.md)
 * [Textarea field](docs/fields/textarea.md)
 * [Select field](docs/fields/select.md)
 * [Submit field](docs/fields/submit.md)
 * [Checkbox field](docs/fields/checkbox.md)
 * [Radio field](docs/fields/radio.md)
 * [Static field](docs/fields/static.md)
 * [StaticNoWrap field](docs/fields/staticNoWrap.md)
