Fusion Form
===========

Pure fusion form rendering with afx support!

Development targets 
-------------------

- Form rendering in fusion with afx and data binding 
- Clear separation of automation and magic for understandable results
- Flexibility:
  - Make no assumption about the required markup like classNames
  - Allow to override all automatically assigned attributes
  - Enable form fragments as fusion components 
  - Allow to bind multiple objects to a single form
  - Enable to create custom controls with  
  - Respect form elements that are defined as plain html when rendering __trustedProperties
- Convenience:
  - Render hidden fields for validation, security and persistence magic
  - Provide validation-errors and restore previously submitted values
  - Prefix fieldnames with the current request namespace if needed 
- Make writing of fusion backend modules easy:  
  - Create a backend field container with translated labels and error messages 
  - Adjust field markup inside the field container for the neos-backend
  
**Important Deviations from Fluid Form ViewHelpers**

The following deviations are probably the ones fluid developers will 
stumble over. There are many more deviations but those are breaking 
concept changes you should be aware of.

- Instead of binding a single `object` a `data` DataStructure is bound to the form.
- Form data-binding is defined via `form.data.object={object}` instead of `objectName` and `object`.
- Field data-binding is defined vis `field.name="object[title]"` with the object name and square brackets for nesting.
- Data binding with `property` path syntax is not supported.
- Select options and groups are defined directly as afx `content` and not `options`.

Usage
-----

Forms usually are defined by using the `Neos.Fusion.Form:Form` prototype 
in afx. The `form.target` can be passed as a string but since it is 
predefined as a `Neos.Fusion:UriBuilder` so in most cases only the target 
`form.target.action` has to be defined. The current `package` and
`controller` are assumed automatically. By default the `form.method` is `post` but 
other methods can be used aswell. 
 
```
renderer = afx`
    <Neos.Fusion.Form:Form form.target.action="sendOrder" form.target.controller="Order" >

    </Neos.Fusion.Form:Form>
`
```

Since forms are used to manipulate existing data those objects or data structures 
are bound the form as `form.data`. The fusion forms allow to manipulate multiple 
objects at once that are sent to the target controller as separate arguments. 

```
renderer = afx`
    <Neos.Fusion.Form:Form form.data.customer={customer} form.data.shipmentAddress={shipmentAddress}>

    </Neos.Fusion.Form:Form>
`
```

The actual input elements, fieldsets and labels are defined as afx content
for the form.

```
renderer = afx`
    <Neos.Fusion.Form:Form form.data.customer={customer}>
        <fieldset>
            <legend for="example">Example</legend>
            <input type="text" name="title" />
        </fieldset>
    </Neos.Fusion.Form:Form>
`
```

While html inputs can be used they provide no magic like data-binding and 
automatic namespaces. 

To render controls that access the data bound to the form prototypes like 
`Neos.Fusion.Form:Input` are used. Those prototyoes are derived from 
`Neos.Fusion.Form:Component.Field` which is responsible for establishing 
the relation between form and field. 

There are plenty of different fieldTypes already that can be found in the 
[Neos.Neos.Form Fusion Documentation](Documentation/FusionReference.rst) but 
it is also easily possible to create new input-types for project specific
purposes.

```
renderer = afx`
    <Neos.Fusion.Form:Form form.data.customer={customer}>
        <Neos.Fusion.Form:Input field.name="customer[firstName]" />
        <Neos.Fusion.Form:Input field.name="customer[lastName]" />
        <Neos.Fusion.Form:Button >Submit</Neos.Fusion.Form:Button>
    </Neos.Fusion.Form:Form>
`
```

It is possible to create field components with translated label and error 
rendering. The prototype `Neos.Fusion.Form:Neos.BackendModule.FieldContainer` 
is an example for that which implements the required markup for Neos backend modules.
Label are added and translated using the translations from `Neos.Neos:Main` 
and validation errors are translated using the source `Neos.Flow:ValidationErrors` 
as translation source. 

ATTENTION: `Neos.Fusion.Form:Neos.BackendModule.FieldContainer` should 
not be used for frontend code and is not designed to be adjustable at all. 
Instead use it as a blueprint to create custom versions that implement 
your project specific markup. 

```
renderer = afx`
    <Neos.Fusion.Form:Form form.data.customer={customer}>
        <Neos.Fusion.Form:Neos.BackendModule.FieldContainer field.name="customer[firstName]">
            <Neos.Fusion.Form:Input />
        </Neos.Fusion.Form:Neos.BackendModule.FieldContainer>
        <Neos.Fusion.Form:Button >Submit</Neos.Fusion.Form:Button>
    </Neos.Fusion.Form:Form>
`
```

Fusion prototypes
-----------------

The full fusion documentation can be found [here](Documentation/FusionReference.rst)

Examples
--------

**Frontend**

This example defines a ShipmentForm component that accepts the props `customer`,
`shipment` and `targetAction`. The form content is defined as afx. The form data is defined 
with both values `customer` and `shipment` that are both modified and sent to the target
controller.

```
prototype(Form.Test:Component.ShipmentForm) < prototype(Neos.Fusion:Component) {
    customer = null
    shipment = null
    targetAction = null
    
    renderer = afx`
        <Neos.Fusion.Form:Form form.data.customer={props.customer} form.data.shipment={props.shipment} form.target.action={props.targetAction} >

            <label for="firstName" >First Name</label>	
            <Neos.Fusion.Form:Input attributes.id="firstName" field.name="customer[lastName]" />
    
            <label for="lastName">Last Name</label>
            <Neos.Fusion.Form:Input attributes.id="lastName" field.name="customer[lastName]" />
    
            <label>Shipment method</label>
            <label><Neos.Fusion.Form:Radio field.name="shipment[method]" field.value="ups" />UPS</label>
            <label><Neos.Fusion.Form:Radio field.name="shipment[method]" field.value="dhl" />DHL</label>
            <label><Neos.Fusion.Form:Radio field.name="shipment[method]" field.value="pickup" />Pickup</label>
    
            <label for="street">Street</label>
            <Neos.Fusion.Form:Input attributes.id="street" field.name="customer[street]" />
            
            <label for="city">City</label>
            <Neos.Fusion.Form:Input attributes.id="city" field.name="customer[city]" />
    
            <label for="country" >Country</label>
            <Neos.Fusion.Form:Select attributes.id=country field.name="shipment[country]">
                <Neos.Fusion.Form:Select.Option option.value="de" >Germany</Neos.Fusion.Form:Select.Option>
                <Neos.Fusion.Form:Select.Option option.value="at" >Austria</Neos.Fusion.Form:Select.Option>
                <Neos.Fusion.Form:Select.Option option.value="ch" > Switzerland </Neos.Fusion.Form:Select.Option>
            </Neos.Fusion.Form:Select>
            
            <Neos.Fusion.Form:Button>Submit Order</Neos.Fusion.Form:Button>

        </Neos.Fusion.Form:Form>
    `
}
```

**Backend Module**

Forms for backend modules basically work the same as in the frontend 
but the additional prototype `Neos.Fusion.Form:Neos.BackendModule.FieldContainer` 
can be used to render fields with translated labels and error messages
using the default markup of the Neos backend.

HINT: To render a backend module with fusion you have to set the 
`defaultViwObjectName` to the `Neos\Fusion\View\FusionView::class` in the
controller class and be aware that you have to include all required fusion
explicitly.

```
#
# In backend modules all includes have to be done explicitly
# the default fusion from Neos.Fusion and Neos.Fusion.Form 
# is needed for rendering of basic forms
#
include: resource://Neos.Fusion/Private/Fusion/Root.fusion
include: resource://Neos.Fusion.Form/Private/Fusion/Root.fusion

#
# By default the fusion view creates the render pathes from the
# current package controller and action. 
#
Form.Test.FusionController.index = Form.Test:Backend.UserForm
Form.Test.FusionController.updateUser = Form.Test:Backend.UserForm

#
# The rendering of the form is centralized in a single prototype 
# that expects the values `title`, `user` and `targetAction` in the context
#
prototype(Form.Test:Backend.UserForm) < prototype(Neos.Fusion:Component) {

    renderer = afx`
        <h2>{title}</h2>

        <Neos.Fusion.Form:Form form.data.user={user} form.target.action={targetAction} >

            <Neos.Fusion.Form:Neos.BackendModule.FieldContainer field.name="user[firstName]" label="user.firstName">
                <Neos.Fusion.Form:Input />
            </Neos.Fusion.Form:Neos.BackendModule.FieldContainer>

            <Neos.Fusion.Form:Neos.BackendModule.FieldContainer field.name="user[firstName]" label="user.lastName">
                <Neos.Fusion.Form:Input />
            </Neos.Fusion.Form:Neos.BackendModule.FieldContainer>

            <Neos.Fusion.Form:Neos.BackendModule.FieldContainer field.name="user[roles]" label="user.role" field.multiple>
                <Neos.Fusion.Form:Checkbox field.value="Neos.Neos:RestrictedEditor" />
                <Neos.Fusion.Form:Checkbox field.value="Neos.Neos:Editor" />
                <Neos.Fusion.Form:Checkbox field.value="Neos.Neos:Administrator" />
            </Neos.Fusion.Form:Neos.BackendModule.FieldContainer>

            <Neos.Fusion.Form:Neos.BackendModule.FieldContainer field.name="user[language]" label="user.language" >
                <Neos.Fusion.Form:Select>
                    <Neos.Fusion.Form:Select.Option option.value="en" >Englisch</Neos.Fusion.Form:Select.Option>
                    <Neos.Fusion.Form:Select.Option option.value="de" >Deutsch</Neos.Fusion.Form:Select.Option>
                    <Neos.Fusion.Form:Select.Option option.value="ru" >Russian</Neos.Fusion.Form:Select.Option>
                    <Neos.Fusion.Form:Select.Option option.value="kg" >Klingon</Neos.Fusion.Form:Select.Option>
                </Neos.Fusion.Form:Select>
            </Neos.Fusion.Form:Neos.BackendModule.FieldContainer>

            <Neos.Fusion.Form:Button>submit</Neos.Fusion.Form:Button>

        </Neos.Fusion.Form:Form>
    `
}
```

Extending Neos.Fusion-Form:
---------------------------

**Custom Form Fields**

The most obvious extension point is the definition of custom fieldtypes.
To do so you have to extend the `Neos.Fusion.Form:Component.Field` prototype
and implement the renderer you need. 

For the rendering you have access to the `field` in the fusion context gives
you access to the current value from data binding and the target value.

HINT: By default all field components support setting `attributes` which
are expected to override all autmatically assigned attributes and whenever 
it makes sense also `content` which is usually defined via afx.  

```
prototype(Neos.Fusion.Form:Textarea)  < prototype(Neos.Fusion.Form:Component.Field) {
    renderer = afx`
        <textarea
            name={field.name}
            {...props.attributes}
        >
            {field.getCurrentValueStringified() || props.content)}
        </textarea>
    `
}
```

**Custom Container with translated labels and errors**

A custom field container is a component that renders label and errors for
a field but expects the field itself as afx content. This pattern allows
to centralize error and label rendering while the field-controls are still 
decided for each field separately.

```
prototype(Vendor.Site:Form.FieldContainer)  < prototype(Neos.Fusion:FieldComponent) {

    name = null
    multiple = false
    label = null
    content = null

    renderer = afx`
        <div class={field.errors ? "error"}>
            <label for={field.name} @if.has={props.label}>
                {I18n.translate(props.label, props.label, [], 'Main', 'Vendor.Site')}
            </label>
            
           {props.content}
            
            <ul @if.hasErrors={field.errors} class="errors">
                <Neos.Fusion:Loop items={field.result.flattenedErrors} itemName="errors" >
                    <Neos.Fusion:Loop items={errors} itemName="error" >
                        <li>
                            {I18n.translate(error.code, error, [], 'ValidationErrors', 'Vendor.Site')}
                        </li>
                    </Neos.Fusion:Loop>
                </Neos.Fusion:Loop>
            </ul>
        </div>
    `

    #
    # all FieldComponents will render the field.name as id so
    # the label for from the FieldContainer references them correctly 
    #
    prototype(Neos.Fusion.Form:Component.Field) {
        attributes.id = ${field.name}
    }
}
```
