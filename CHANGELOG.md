# praxis-blueprints changelog

## next

## 3.4

* Introduce an overridable `_cache_key` for Blueprints. Default it to the strict `Object` reference (instead of the `.object_id` key used before, to avoid GC re-assigning previously rendered objects)

## 3.3

* Include Attributor::Dumpable in Blueprint, so it renders (semi) correctly if
  rendered with just `true` specified for fields.
* Fix bug rendering subobjects with nil values (manifested when `include_nil: true` there’s an explicit subsection of fields)

## 3.2

* Ensure we call `object.dump` in Renderer when fully dumping an instance (or array of instances) that have the Attributor::Dumpable module (i.e., when no subfields were selected)
  * In other words, attributor types (custom or not) will need to include the Attributor::Dumpable module and properly implement the dump instance method to produce the right output with native ruby objects.
* Ensure `Blueprint` validates advanced requirements.

## 3.1

* Reworked FieldExpander's behavior with circular references.
  * Removed `Praxis::FieldExpander::CircularExpansionError`
  * `FieldExpander#expand` now returns self-referential hashes for circular
    expansions
* `Renderer#render` catches stack overflows and returns a
  `Praxis::Rendering::CircularRenderingError`, which includes a portions
  from the start and end of the context at the time of the exception.
* Report `anonymous` in Blueprints following `Attributor` semantics.

## 3.0

* Added `FieldExpander`, a new class that recursively expands a tree of
  of attributes for Blueprint types as well as view objects into their lowest-level values.
    * For example, for the `Person` and `Address` blueprints defined in
    [spec_blueprints.rb](sped/support/spec_blueprints.rb):
      * `FieldExpander.expand(Person, {name: true, full_name: true})` would return
        `{name: true, full_name: {first: true, last:true}}`.
    * Attempting to expand attributes with circular references (i.e. the
      a person's address, where that address then references that person) will
      raise a `Praxis::FieldExpander::CircularExpansionError` error.
* Added `Renderer`, a new class for rendering objects (typically Blueprint
  instances) with an explicit and recursive set of fields (as from
  `Fieldexpander`).
    * For example:
      * `render(person, name: true)` would produce `{name: person.name}`
      * `render(person, name: true, address: {state: true})` would
        produce `{name: person.name, address: {state: person.address.state}}`
    * Accepts an `include_nil:` option when initialized, that if true will
      cause the renderer to output values that are nil, rather than omit them.
      Note: this will apply to the entire rendering, there is no way to set it
      for only a portion of the rendering, or on a per-View basis.
* Added `Blueprint.domain_model` to specify the underlying domain model, e.g.
  a `Praxis::Mapper::Resource`. Accepts a string value that is resolved when
  `Blueprint.finalize!` is called. Defaults to `Object`.
* `Blueprint.load` will use the `domain_model` to wrap the incoming object
  if it is not an instance of that class. Note: this does not change existing
  default if `domain_model` is not set.
* Refactored `View` and `CollectionView` rendering to use a `Renderer` with
  a set of expanded fields.
* The automatically generated `:master` view will now render sub-attributes
  using their `:default` view, rather than `:master`.
* `Blueprint#render` now uses `Renderer` and `FieldExpander` internally. In most
  cases this will continue to function identically to the old behavior, but
* Views defined with `Blueprint.view`, including subviews defined inside,
  no longer accept the `:include_nil` option. This can be set using the same
  option on `Renderer` described above.


## 2.2

* Added instrumentation through `ActiveSupport::Notifications`
  * Use the 'praxis.blueprint.render to subscribe to `Blueprint#render` calls


## 2.1

* Fixed handling of objects passed in to `Blueprint.example`.
* Bumped attributor dependnecy to >= 4.0.1


## 2.0.1

* relaxed attributor dependency to >= 3.0


## 2.0

* Fix `Blueprint.new` handling of caching when the wrapped object responds to `identity_map`, but does not have one set.
* Undefine JRuby package helper methods in `Model` (org, java...)
* Added support for rendering custom views
  * Internally, a `View` object can now be dumped passing a `:fields` option (which is a hash, that can recursively will define which sub-attributes to render along the way). See [this spec](https://github.com/rightscale/praxis-blueprints/blob/master/spec/praxis-blueprints/blueprint_spec.rb) for an example.
  * `Blueprints` will also accept the `:fields` option (with the same hash syntax), but it will also accept an array to imply the list of top-level attributes to render (when recursion is not necessary)
  * Caching of rendered blueprints will continue to work if the view is re-rendered with equivalent `:fields`
* Deprecate `Blueprint.render(view_name,...) positional param
  * Please use :view named parameter instead. I.e., `render(:default, context: ...)`  => `render(view: :default, context: ...)`

## 1.3.1

* Improve error for nonexistent view attributes in media type
* Added `family` method to Blueprints to follow the new `Attributor` practices.

## 1.3.0

* Added `include_nil` option to `View` for refining the rendering of nil values.
  * This can be set when defining the view with `Blueprint.view`, and defaults to false. For example: `view :default, include_nil: true { ... }`.
* Fixed `Blueprint.describe` to output the proper `:id`. That is: the 'id` from the Class name, rather than the internal Struct attribute.
* Fixed `Blueprint.dump(nil)` raising a `NoMethodError`
* Fixed `Blueprint.load(nil)` to properly support the `recurse` option.

## 1.2.0

* `Blueprint` readers now always `load` the value for an attribute before returning it.

## 1.1.0

* Added CollectionView, a special type of view that renders an array of objects with another view.


## 1.0.1

* Relaxed ActiveSupport version dependency (from 4 to >=3)


## 1.0

Initial release!
