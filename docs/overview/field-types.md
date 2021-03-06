---
order: 8
---

Common Field Types
==================

Below are the most common field types used when [defining a data model] for use with wq.  You may also be interested in the full lists of [XLSForm question types] and [Django field types].

If you want to define nested / repeat forms or allow user-defined fields, see [Advanced Patterns]. 

## Text Fields

### Short Text Input (Char)

<ul data-role="listview" data-inset="true">
  <li class="ui-field-contain">
    <label for='input_types-char_field'>Char field</label>
    <input id='input_types-char_field' type='text' data-xform-type='string' name='char_field' value="">
    <p class="hint">Enter some text.</p>
    <p class='error input_types-char_field-errors'></p>
  </li>
</ul>

*XLSForm Definition*:

type | name | label | hint | required | constraint
-----|------|-------|------|----------|------------
string | [name] | Char field | Enter some text. | | wq:length(5)

*Django definition:*

```python
from django.db import models

class MyModel(models.Model):
    [name] = models.CharField(
        max_length=5,
        null=True,
        blank=True,
        verbose_name="Char field",
        help_text="Enter some text.",
    )
```

### Long Text Input (Text)

<ul data-role="listview" data-inset="true">
  <li class="ui-field-contain">
    <label for='input_types-text_field'>Text field</label>
    <textarea id='input_types-text_field' name='text_field' data-xform-type="text"></textarea>
    <p class="hint">Enter some text.</p>
    <p class='error input_types-text_field-errors'></p>
  </li>
</ul>

*XLSForm Definition:*

type | name | label | hint | required | constraint
-----|------|-------|------|----------|------------
text | [name] | Text field | Enter some text. | | 

*Django definition:*

```python
from django.db import models

class MyModel(models.Model):
    [name] = models.TextField(
        null=True,
        blank=True,
        verbose_name="Text field",
        help_text="Enter some text.",
    )
```

### Note (XLSForm Only)

<ul data-role="listview" data-inset="true">
  <li data-xform-type='note'>
    <p class="label">This is a note.</p>
  </li>
</ul>

*XLSForm Definition:*

type | name | label | hint | required | constraint
-----|------|-------|------|----------|------------
note | [name] | This is a note. | | | 

*Django definition:*

Django models do not have a concept corresponding to XLSForm's "note" type.  However, it is trivial to edit the HTML form generated by wq to add any formatting or custom text you need.

## Numeric Fields

### Integer

<ul data-role="listview" data-inset="true">
  <li class="ui-field-contain">
    <label for='input_types-int_field'>Integer field</label>
    <input id='input_types-int_field' type='number' data-xform-type='integer' name='int_field' value="">
    <p class="hint">Enter an integer number.</p>
    <p class='error input_types-int_field-errors'></p>
  </li>
</ul>

*XLSForm Definition:*

type | name | label | hint | required | constraint
-----|------|-------|------|----------|------------
integer | [name] | Integer field | Enter an integer number. | | 

*Django definition:*

```python
from django.db import models

class MyModel(models.Model):
    [name] = models.IntegerField(
        null=True,
        blank=True,
        verbose_name="Integer field",
        help_text="Enter an integer number.",
    )
```

### Decimal / Float

<ul data-role="listview" data-inset="true">
  <li class="ui-field-contain">
    <label for='input_types-dec_field'>Decimal field</label>
    <input id='input_types-dec_field' type='number' data-xform-type='decimal' name='dec_field' step='0.001' value="">
    <p class="hint">Enter a decimal number.</p>
    <p class='error input_types-dec_field-errors'></p>
  </li>
</ul>

*XLSForm Definition:*

type | name | label | hint | required | constraint
-----|------|-------|------|----------|------------
decimal | [name] | Decimal field | Enter a decimal number. | | 

*Django definition:*

```python
from django.db import models

class MyModel(models.Model):
    [name] = models.FloatField(
        null=True,
        blank=True,
        verbose_name="Decimal field",
        help_text="Enter a decimal number.",
    )
```

> FIXME: Django also has a separate `DecimalField`, should probably use that instead?

## Choice (Domain) Fields

### Static Choices

Static choice fields are useful for quick survey questions with a small set of rarely changing options.  "Yes/No" fields can also be implemented this way (though Django also includes a separate `BooleanField`).

<ul data-role="listview" data-inset="true">
  <li class="ui-field-contain">
    <fieldset data-xform-type='select one' data-role='controlgroup' data-type='horizontal'>
      <legend>Pick a color</legend>
      <input type='radio' id='select-color-red' name='color' value='red'>
      <label for='select-color-red'>Red</label>
      <input type='radio' id='select-color-green' name='color' value='green'>
      <label for='select-color-green'>Green</label>
      <input type='radio' id='select-color-blue' name='color' value='blue'>
      <label for='select-color-blue'>Blue</label>
    </fieldset>
    <p class='error select-color-errors'></p>
  </li>
</ul>

*XLSForm Definition:*

**survey tab**

type | name | label | hint | required | constraint
-----|------|-------|------|----------|------------
select_one colors | [name] | Color | Pick a color | | 

**choices tab**

list name | name | label
----------|------|-------
colors | red  | Red
colors | green | Green
colors | blue | Blue

*Django definition:*

```python
from django.db import models

class MyModel(models.Model):
    [name] = models.CharField(
        choices=(
            ("red", "Red"),
            ("green", "Green"),
            ("blue", "Blue"),
        ),
        max_length=5,
        null=True,
        blank=True,
        verbose_name="Pick a color",
    )
```

### Dynamic Choices (ForeignKey)

Dynamic choices make it possible for any user of your application to dynamically update the available domain values without you needing to recompile the application.  Under the hood, dynamic choices are implemented as foreign keys to other existing relational tables.  In fact, wq does not distinguish in any meaningful way between tables used as domain values versus tables used to manage the actual observation data.  This means all tables can be registered with the REST API and managed from the client app (assuming that the appropriate permissions are given to each respective user.)  For example, one of the most common schema designs has been to split the observation timeseries into separate "Site" and "Observation" tables, with a ForeignKey pointing from Observation to Site.  Users then can manage their list of sites separately from their observational data.

As of wq version 1.0, it is possible to use foreign keys to link parent-child records while working offline, even when the parent record has not yet been synced to the server.  The example below assumes that "Site A3" was created on a separate form that has not yet been synced to the server.  If that site was selected when this form was saved, [wq/outbox.js] would ensure that Site A3 is properly synced before attempting to sync this form.

> Note: If you need your dynamic choice list to work offline, be sure to set `cache="all"` when registering the domain model with the router.  See the [configuration documentation][config] and the example below.

Depending on your use case, it is also possible to define a single form with nested children that populates multiple tables at once - see [Advanced Patterns] for more information.

<ul data-role="listview" data-inset="true">
  <li class="ui-field-contain">
    <label for='select-site_id'>Pick a Site ID</label>
    <select id='select-site_id' data-xform-type='integer' name='site_id' required>
      <option value="">Select one...</option>
      <option value="site-A3">* Site A3</option>
      <option value="landfill">Landfill Site</option>
      <option value="cd2">CD2</option>
      <option value="cd3">CD48</option>
      <option value="creek">Pine Creek Bridge</option>
    </select>
    <p class='error select-site_id-errors'></p>
  </li>
</ul>

*XLSForm Definition:*

type | name | label | hint | required | constraint
-----|------|-------|------|----------|------------
integer | [name] | Pick a Site ID | wq:ForeignKey("other_app.Site") | yes |

> This is a wq-specific extension to the XLSForm syntax.  It assumes that `other_app.Site` has already been defined in another Django app.  If you would like to define multiple tables in the same XLSForm, see [Advanced Patterns].

*Django definition:*

```python
from django.db import models

class MyModel(models.Model):
    [name] = models.ForeignKey(
        "other_app.Site",
        verbose_name="Pick a Site ID",
    )
```

Note: in order to ensure the full site list is available offline, `other_app.Site` should be [configured][config] with `cache="all"`:

```python
# other_app/rest.py
from wq.db import rest
from .models import Site

rest.router.register_model(
    Site,
    fields="__all__",
    cache="all",
)
```

## Date & Time Fields

### Date

<ul data-role="listview" data-inset="true">
  <li class="ui-field-contain">
    <label for='input_types-date_field'>Date field</label>
    <input id='input_types-date_field' type='date' data-xform-type='date' name='date_field' value="">
    <p class="hint">Enter a date.</p>
    <p class='error input_types-date_field-errors'></p>
  </li>
</ul>

*XLSForm Definition:*

type | name | label | hint | required | constraint
-----|------|-------|------|----------|------------
date | [name] | Date field | Enter a date. | | 

*Django definition:*

```python
from django.db import models

class MyModel(models.Model):
    [name] = models.DateField(
        auto_now_add=True,
        verbose_name="Date field",
        help_text="Enter a date.",
    )
```

### Time

<ul data-role="listview" data-inset="true">
  <li class="ui-field-contain">
    <label for='input_types-time_field'>Time field</label>
    <input id='input_types-time_field' type='time' data-xform-type='time' name='time_field' value="">
    <p class="hint">Enter a time.</p>
    <p class='error input_types-time_field-errors'></p>
  </li>
</ul>

*XLSForm Definition:*

type | name | label | hint | required | constraint
-----|------|-------|------|----------|------------
time | [name] | Time field | Enter a time. | | 

*Django definition:*

```python
from django.db import models

class MyModel(models.Model):
    [name] = models.TimeField(
        auto_now_add=True,
        verbose_name="Time field",
        help_text="Enter a time.",
    )
```

### Date + Time

<ul data-role="listview" data-inset="true">
  <li class="ui-field-contain">
    <label for='input_types-datetime_field'>Date+time field</label>
    <input id='input_types-datetime_field' type='datetime-local' data-xform-type='dateTime' name='datetime_field' value="">
    <p class="hint">Enter a date and a time.</p>
    <p class='error input_types-datetime_field-errors'></p>
  </li>
</ul>

*XLSForm Definition:*

type | name | label | hint | required | constraint
-----|------|-------|------|----------|------------
dateTime | [name] | Date+time field | Enter a date and a time. | | 

*Django definition:*

```python
from django.db import models

class MyModel(models.Model):
    [name] = models.DateTimeField(
        auto_now_add=True,
        verbose_name="Date+time field",
        help_text="Enter a date and a time.",
    )
```

## Files & Photos

### Photo

<ul data-role="listview" data-inset="true">
  <li class="ui-field-contain">
    <img src="https://wq.io/images/empty.png"
         id="input_types-image_field-preview">
    <label for="input_types-image_field">Photo</label>
    <input type="file" name="image_field" id="input_types-image_field" accept='image/*'
           data-wq-preview="input_types-image_field-preview">
    <p class="error input_types-image_field-errors"></p>
  </li>
</ul>

*XLSForm Definition:*

type | name | label | hint | required | constraint
-----|------|-------|------|----------|------------
image | [name] | Image field | Add an image. | yes | 

*Django definition:*

```python
from django.db import models

class MyModel(models.Model):
    [name] = models.ImageField(
        upload_to="[folder name]",
        verbose_name="Image field",
        help_text="Add an image.",
    )
```

> This field uses a wq/app.js plugin to display the image preview.  For more information, see the documentation for [wq/photos.js].

## Geospatial Fields

### Point

<ul data-role="listview" data-inset="true">
  <li>
    <label for='input_types-point_field'>Point field</label>
    <input type='hidden' data-xform-type='geopoint' name='point_field' required>
    <div class="map edit-map" id='point-map' data-interactive style='height:300px;background:#ccc;border:1px solid black'></div>
    <p class="hint">Enter a point.</p>
    <p class='error input_types-point_field-errors'></p>
  </li>
</ul>

*XLSForm Definition:*

type | name | label | hint | required | constraint
-----|------|-------|------|----------|------------
geopoint | [name] | Point field | Enter a point. | yes | 

*Django definition:*

```python
from django.contrib.gis.db import models

class MyModel(models.Model):
    [name] = models.PointField(
        srid=4326,
        verbose_name="Point field",
        help_text="Enter a point.",
    )
```

> This field uses a wq/app.js plugin to display the map editor.  For more information, see the documentation for [wq/map.js].  If you are collecting point locations via GPS, you may also be interested in the [wq/locate.js] plugin.

### LineString

<ul data-role="listview" data-inset="true">
  <li>
    <label for='input_types-linestring_field'>Line string field</label>
    <input type='hidden' data-xform-type='geotrace' name='linestring_field'>
    <div class="map edit-map" id='linestring-map' data-interactive style='height:300px;background:#ccc;border:1px solid black'></div>
    <p class="hint">Enter a line.</p>
    <p class='error input_types-linestring_field-errors'></p>
  </li>
</ul>

*XLSForm Definition:*

type | name | label | hint | required | constraint
-----|------|-------|------|----------|------------
geotrace | [name] | Line string field | Enter a line. | | 

*Django definition:*

```python
from django.contrib.gis.db import models

class MyModel(models.Model):
    [name] = models.LineStringField(
        srid=4326,
        null=True,
        blank=True,
        verbose_name="Line string field",
        help_text="Enter a line.",
    )
```

> This field uses a wq/app.js plugin to display the map editor.  For more information, see the documentation for [wq/map.js].

### Polygon

<ul data-role="listview" data-inset="true">
  <li>
    <label for='input_types-polygon_field'>Polygon field</label>
    <input type='hidden' data-xform-type='geoshape' name='polygon_field'>
    <div class="map edit-map" id='polygon-map' data-interactive style='height:300px;background:#ccc;border:1px solid black'></div>
    <p class="hint">Enter a polygon.</p>
    <p class='error input_types-polygon_field-errors'></p>
  </li>
</ul>

*XLSForm Definition:*

type | name | label | hint | required | constraint
-----|------|-------|------|----------|------------
geoshape | [name] | Polygon field | Enter a polygon. | | 

*Django definition:*

```python
from django.contrib.gis.db import models

class MyModel(models.Model):
    [name] = models.PolygonField(
        srid=4326,
        null=True,
        blank=True,
        verbose_name="Polygon field",
        help_text="Enter a polygon.",
    )
```

> This field uses a wq/app.js plugin to display the map editor.  For more information, see the documentation for [wq/map.js].

[defining a data model]: https://wq.io/docs/data-model
[XLSForm question types]: http://xlsform.org/#question-types
[Django field types]: https://docs.djangoproject.com/en/1.10/ref/models/fields/#field-types
[Advanced Patterns]: https://wq.io/docs/nested-forms
[wq/photos.js]: https://wq.io/docs/photos-js
[wq/map.js]: https://wq.io/docs/map-js
[wq/locate.js]: https://wq.io/docs/locate-js
[wq/outbox.js]: https://wq.io/docs/outbox-js
[config]: https://wq.io/docs/config
