
# What is all about #

swinghtmltemplate let's you construct gui forms by declaring theirs tamplate using html-like syntax.

Templates are stored in external file with ".html" extension, they are loaded in runtime and are coverted to
swing-components. You can assign these components to some class fields using `@Bind` annotaion during
conversion process.

Also you can bind methods calls to component events (e.g. button click) and sync model properties with
components values.

# Installation #
Add the following too your pom.xml:
```
    <dependencies>
        <dependency>
            <groupId>com.googlecode.swinghtmltemplate</groupId>
            <artifactId>swinghtmltemplate</artifactId>
            <version>0.4</version>
        </dependency>
    </dependencies>


    <repositories>
        <repository>
            <id>swinghtmltemplate</id>
            <name>Swing html template repo</name>
            <url>http://swinghtmltemplate.googlecode.com/svn/maven-repository</url>
            <releases>
                <enabled>true</enabled>
                <updatePolicy>never</updatePolicy>
            </releases>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
    </repositories>

```


# How does this work #

Loading of html-document is handled by [jdom](http://www.jdom.org/) library, so html must be valid xml.

Next, dom-model is converted to the tree of tags, ready to produce swing components. After that
tree of swing component is being built.

# Usage #
Loading of dom-model is done with
```
InputStream in = ...;
DomModel model = DomLoader.loadModel(in);
```

To built swing-components tree use
```
DomConverter.toSwing(model);
```

You can self-create some components and pass them to converter, so these components will be used
as corresponding tag components, rather than creating new ones:
```
JLabel rootLabel = new JLabel("Foo");

String html = "<html>" +
        "<head>" +
        "<style>" +
        "p {" +
        "   type: text;" +
        "}" +
        "</style>" +
        "</head>" +
        "<body style='display: border;'>" +
        "   <p type='html' id='rootLabel'>center</p>" +
        "   <p align='top'>top</p>" +
        "   <p align='bottom'>bottom</p>" +
        "   <p align='left'>left</p>" +
        "   <p align='right'>right</p>" +
        "</body>" +
        "</html>";

//create substitutions map
Map<Selector, JComponent> substitutions = new HashMap<Selector, JComponent>();
//add substitution for tag with id='rootLabel'
substitutions.put(new Selector("#rootLabel"), rootLabel);

DomModel model = DomLoader.loadModel(new ByteArrayInputStream(html.getBytes()));
DomConverter.toSwing(model, substitutions);
Tag body = model.getRootTag().getChildByName("body");

JPanel rootPanel = (JPanel) body.getComponent();
BorderLayout l = (BorderLayout) rootPanel.getLayout();
JLabel centerLabel = (JLabel) l.getLayoutComponent(BorderLayout.CENTER);

assertEquals(centerLabel, rootLabel);
```


next you must get root swing component from body tag:
```
model.getRootTag().getChildByName("body").getComponent()
```
root swing element is associated with body tag.

Also you can get any tag by it's id:
```
model.getTagById("someId");
```

## Auto binding of components ##
You can automatically load dom-model for some object (which is a controller for some form) and assign
swing components to object's fields:
```
package foo;

public class LoginForm {

   @Bind("login")
   private JTextField loginText;

   public LoginForm() {
        try {
            Binder.bind(this);
        } catch (JDOMException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }

   }

}
```

`Binder.bind(this)` loads html document, located at the same place, where class is located and which has
the same name, for an example above the full name would be `"/foo/LoginForm.html"`

`@Bind("login")` annotation tells, that the field, marked with this annotation will be assigned
a swing component, associated with the tag with id, equals to the value of the annotation ("login" in the example).

The object `LoginForm` plays role of controller here. We call it controller.

Also, you can make controller a root component of dom-model. The controller must extend `JComponent`:
```
package foo;

public class LoginForm extends JPanel {

   @Bind("login")
   private JTextField loginText;

   public LoginForm() {
        try {
            //pass 'true' to use LoginForm as root compponent
            Binder.bind(this, true);
        } catch (JDOMException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }

   }

}
```

## Binding model elements ##
Dom model can contain some elements, which are exposed to the components. To add model element you have
2 ways:

  * call addModelElement method:
```
DomModel model = ...;
Foo foo = ...;
model.addModelElement("foo", foo);
DomConverter.convert(model);
```
Remember to add model elements before converting dom model to swing.

  * annotate properties on controller (when using Binder.bind()) with @ModelElement:
```
package foo;

public class LoginForm {

   @ModelElement("account")
   private Account account;

   public LoginForm() {
        try {
            Binder.bind(this);
        } catch (JDOMException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }

   }

}


class Account {
    private String login = "";
    //getter and setter
}
```
Account object will be added as model element under the `account` key.

Components values can be synced with model elements (if they support this).
These components support binding:

  * `<input type='text'/>`
  * `<input type='password'/>`
  * `<input type='checkbox'/>`
  * `<spinner/>`
  * `<textarea/>`
You can bind component to model element using `value` attribute. Specify EL expression for the needed property:
```
<html>
<body>
    <input type='text' value='${account.login}'/>
</body>
</html>
```

Binding is done with [betterbeansbinding](http://kenai.com/projects/betterbeansbinding/pages/Home).


## Selecting component with selectors ##
You can select components from dom-model using selector:
```
InputStream in = ...
DomModel model = DomLoader.loadModel(in);
for (JComponent c : model.select(".button")) {
   c.setEnabled(false);
}
```

# Components #

All tags are splitted into libraries, each one identified by namespace.
All html tags are located at default library with empty namespace. They are also available in
'http://www.w3.org/1999/xhtml' namespace library. These html markups will produce identical results:
```
<html xmlns="http://www.w3.org/1999/xhtml">
<body>
  <p>Foo</p>
</body>
</html>
```

```
<html>
<body>
  <p>Foo</p>
</body>
</html>
```

## Common attributes ##
For any component you can assign it's properties by providing attributes in associated tag or by specifying
css styles. Below are common properties (for all tags):

  * `id` - component identifier
  * `display` - identifier of the layout manager for the component, associated with tag. Defaults to `flow (java.awt.BorderLayout)`), `div` defaults to `border (java.awt.BorderLayout)`).
  * `align` - this is constraint for this component in parent's layout manager. Each layout manager has corresponding support-class, which role is to convert text values of this attribute to suitable values of layout manager. For example, support class for `java.awt.BorderLayout` converts strings `top`, `bottom`, `left`, `right`, `center` into `BorderLayout.NORTH`, `BorderLayout.SOUTH` etc accordingly
  * `text-align` - layout specific. Used to align on left/right side (but can be ignored, if layout manager doesn't supports aligning)
  * `vertial-align` - layout specific. Used to align on top/bottom side (but can be ignored, if layout manager doesn't supports aligning)
  * `type` - depends on tag. Used to specify the type of component to produce. Examples: `<input type="text" /> -> JTextField`, `<input type="button" /> -> JButton`
  * `width` - component's width, layout specific
  * `height` - component's height, layout specific
  * `style` - this is treated as properties list, but in form of css
  * `text` - text of the component. Method `setText(String)` is called for this property. If there's no such methos, warning is printed to log.
  * `opaque` - opacity. Same as `JComponent.opaque`. Values: `true` or `false`
  * `icon` - component's icon. Method `setIcon(Icon)` is called for this property. If there's no such methos, warning is printed to log. Value - icons path (icon must be available for  `getClass().getResource("url"))`.
  * `border` - component's border. Value - border description (see border section for details)
  * `enabled` - enables or disables component
  * `color` - component's color. Method `JComponent.setForeground(Color)` is called for this property
  * `background-color` - component's background color. Method `JComponent.setBackground(Color)` is called for this property
  * `font-size` - component's font size in pixels.
  * `font-weight` - component's font weight. Default's to default component style (JLabel prints text in bold, for example). Values: `normal`, `bold`, `bolder`, `lighter`.
  * `font-style` - component's font style. If value is `italic`, font will be italic.
  * `font-family` - component's font family.


## default namespace ##

### body ###
Tag is converted to `javax.swing.JPanel`. `java.awt.FlowLayout` is used as layout manager by default.

### div ###
Tag is converted to `javax.swing.JPanel`. `java.awt.BorderLayout` is used as layout manager by default.

### form ###
Tag is converted to `javax.swing.JPanel`. `net.miginfocom.swing.MigLayout` is used as layout manager by default.

### hr ###
Tag is converted to `javax.swing.JSeparator`.
Use  `type="horizontal"` or `type="vertical"` for specifying orientation of separator.

### img ###
Tag is converted to `javax.swing.JLabel`. Value of the `src` attribute is copied to `icon` attribute, so,
an icon, specified as `src` value will be component's icon (see common properties)

Example
```
<img src="/img/login-icon.gif"/>
```

### input ###
According to `type` value converts to
  * `type="text"` - `JTextField`
  * `type="password"` - `JPasswordField`
  * `type="button"` - `JButton`
  * `type="checkbox"` - `JCheckBox`
  * `type="radio"` - `JRadioButton`
Converts to `JTextField` by default. If `type` equals to unknown string, `type` will be resolved to
`text`, warning will be printed to log.

```
<input type="button" text="OK"/>
```


If resulting component is subclass of `javax.swing.text.JTextComponent`, then the contents of the tag will be component's text.
```
<input type="text">Initial text</input>
```

`<input>` with type `text`, `password` and `checkbox` can be binded to the model element property using
value attribute:
```
<input type='text' value='${account.login}'/>
```
text inputs can be binded to `java.lang.String`type properties.
checkboxes can be binded to `boolean` (not `java.lang.Boolean`) type properties.


### object ###
Tag is converted to the component which classname is specified with `classid` attribute.
Example:
```
<object classid="javax.swing.JButton" text="OK"/>
```

### p ###
Tag is converted to `javax.swing.JLabel`. The content of the tag is assigned as a component's text.
If `content` equals to `html`, then the content of the tag is surrounded with `<html>` before and `</html>` after,
so `JLabel` will produce html. If `content` equals to `text` or is empty,
then the tag content is just assigned as a component's text.

Example:
```
<p>Login:</p>
<p content="html"><![CDATA[<u>Login:</u>]]></p>
<p content="text"><![CDATA[<html><u>Login:</u></html>]]></p>
```



### span ###
Tag is converted to `javax.swing.JPanel`. `java.awt.FlowLayout` is used as layout manager by default.

Example:
```
<span style="text-align:right">
   <input type="button" text="OK"/>
   <input type="button" text="Cancel"/>
</span>
```


### table ###
Tag is converted to `javax.swing.JPanel`, with [tableLayout](https://tablelayout.dev.java.net/) as layout manager.

Supported attributes
  * `x-tablelayout-column-sizes` - column sizes in [tableLayout](https://tablelayout.dev.java.net/) manner
  * `x-tablelayout-row-sizes` - row sizes in [tableLayout](https://tablelayout.dev.java.net/) manner
  * `cellspacing` - space between cells in pixels

Example:
```
<table style="x-tablelayout-column-sizes: preferred fill; x-tablelayout-row-sizes: preferred">
   <tr>
      <td><p>Login:</p></td>
      <td><input type="text"/></td>
   </tr>
</table>
```

If you skip `x-tablelayout-column-sizes` (`x-tablelayout-row-sizes`),
then corresponding attribute will be counted dynamicly: maximum width (height) is recorder for each cell
and later is used as column width (row height).

You must remember that
```
preferred < number < fill
```

Default size is `preferred`. So, if you specify `width` (`height`) for any cell with numeric value,
then this value will override `preferred`. Similarly value `fill` overrides any numeric value and `preferred`.

Pay attention that specifying `x-tablelayout-column-sizes` (`x-tablelayout-row-sizes`) automatically
sets column (row) count (so any column outside column count will be invisible).
Column (row) count is cumputed dynamically otherwise.

Tag `td` supports properties:
  * `width` - cells width, see above
  * `height` - cells height, see above
  * `rowspan` - rowspan of the cell. Cells with `rowspan>1` are ignored during dynamic count of row height
  * `colspan` - colspan of the cell. Cells with `colspan>1` are ignored during dynamic count of column width
  * `text-align` - horizontal align. Values: `left`, `right`, `center`, `full`
  * `vertcal-align` - vertical align. Values: `top`, `bottom`, `center`, `full`

Examples:
```
<table style="x-tablelayout-row-sizes: preferred preferred preferred preferred; x-tablelayout-column-sizes: 100 fill; border: compound (etched) (empty 12 12 12 12)">

    <tr>
        <td style="text-align: right;"><p>Login:</p></td>
        <td><input type="text">qqq</input></td>
    </tr>
    <tr>
        <td style="text-align: right;"><p>Password:</p></td>
        <td>
            <input type="text">***</input>
        </td>
    </tr>
    <tr>
        <td style="text-align: right;"><p>Remember:</p></td>
        <td>
            <input type="checkbox"/>
        </td>
    </tr>
    <tr>
        <td style="text-align: right;vertical-align:top"><p>Login as:</p></td>
        <td>
            <table style="x-tablelayout-row-sizes: preferred preferred; x-tablelayout-column-sizes: preferred fill">
                <tr>
                    <td><input type='radio'/></td>
                    <td><p>Manager</p></td>
                </tr>
                <tr>
                    <td><input type='radio'/></td>
                    <td><p>Admin</p></td>
                </tr>
            </table>
        </td>
    </tr>
</table>

<table>
    <tr>
        <td rowspan="3" class="top">
            <img src="/img/login_icon.gif" alt=""/>
        </td>
        <td width="fill">
            <form id="loginForm">
                <p>Login:</p>
                <input id="login" type="text" align="wrap"/>
                <p>Password:</p>
                <input id="password" type="password" align="wrap" />
            </form>
        </td>
    </tr>
    <tr>
        <td>
            <p id="result" class="red"></p>
        </td>
    </tr>
    <tr>
        <td>
            <div style="display: box; x-boxlayout-direction: horizontal;border: empty 0 0 0 6">
                <glue type="horizontal"/>
                <input type="button" text="OK" id="ok" icon="/img/accept.png" />
                <strut type="horizontal" style="width: 6"/>
                <input type="button" text="Cancel" id="cancel" icon="/img/cancel.png"/>
            </div>
        </td>
    </tr>
</table>

```

### textarea ###
Tag is converted to `JTextArea`. Tag's content is used as component's text.

Tag supports properties:
  * value - en EL expression, describing a property of a model element, the text of the component is binded to.

Example:
```
<scroll>
   <textarea>Some text</textarea>
</scroll>
```



## `http://www.oracle.com/swing` namespace ##

### attribute ###
Special tag for assigning value to the parent's tag component. Doesn't convert to any swing component.

Supported attributes for tag:
  * `name` - field name, to which the value will be assigned. Example: `preferredSize`
  * `value` - value to be assigned, as string
  * `type` - field type. Full class name. Example, `java.awt.Dimension`

Supported `value` formats according to `type` value:
  * `type='java.lang.String'` : string, example: `<attribute name='name' value='Foo' type='java.lang.String'/>`
  * `type='boolean'` : `true` or `false`, example: `<attribute name='autoscrolls' value='false' type='boolean'/>`
  * `type='int'` : integer, example: `<attribute name='width' value='500' type='int'/>`
  * `type='long'` : long, example: `<attribute name='uuid' value='50000' type='long'/>`
  * `type='float'` : float, example: `<attribute name='width' value='5.0' type='float'/>`
  * `type='double'` : double, example: `<attribute name='width' value='5.0' type='double'/>`
  * `type='short'` : short, example: `<attribute name='width' value='5' type='short'/>`
  * `type='byte'` : bute, example: `<attribute name='width' value='5' type='byte'/>`
  * `type='char'` : charachter, example: `<attribute name='mnemonic' value='f' type='char'/>`
  * `type='java.lang.Boolean'` : `true` or `false`, example: `<attribute name='autoscrolls' value='false' type='java.lang.Boolean'/>`
  * `type='java.lang.Integer'` : integer, example: `<attribute name='width' value='500' type='java.lang.Integer'/>`
  * `type='java.lang.Long'` : long, example: `<attribute name='uuid' value='50000' type='java.lang.Long'/>`
  * `type='java.lang.Float'` : float, example: `<attribute name='width' value='5.0' type='java.lang.Float'/>`
  * `type='java.lang.Double'` : double, example: `<attribute name='width' value='5.0' type='java.lang.Double'/>`
  * `type='java.lang.Byte'` : byte, example: `<attribute name='width' value='5' type='java.lang.Byte'/>`
  * `type='java.lang.Short'` : short, example: `<attribute name='width' value='5' type='java.lang.Short'/>`
  * `type='java.lang.Charachter'` : charachter, example: `<attribute name='mnemonic' value='f' type='java.lang.Charachter'/>`
  * `type='java.awt.Dimension'` : 2 integers, separated by spaces, example: `<attribute name='preferredSize' value='500 100' type='java.awt.Dimension'/>`
  * `type='java.awt.Insets'` : 4 integers, separated by spaces, example: `<attribute name='insets' value='5 1 3 4' type='java.awt.Insets'/>`
  * `type='java.awt.Point'` : 2 integers, separated by spaces, example: `<attribute name='location' value='5 1' type='java.awt.Point'/>`
  * `type='java.awt.Rectangle'` : 4 integers, separated by spaces, example: `<attribute name='bounds' value='5 1 3 4' type='java.awt.Rectangle'/>`

Pay attention, that methods with primitive types as operands are not equal to methods with objects as operands.
`setFoo(java.lang.Integer)` is not equal to `setFoo(int)`. You must specify correct `type` value.

Example:
```
<html xmlns:c="http://www.oracle.com/swing">
<body>
   <c:attribute name='preferredSize' value='500 100' type='java.awt.Dimension'/>
   <p>Panel has preferred size 500x100px</p>
</body>
</html>
```

### column ###
Special tag for specifying columns in dataTable tag.
Possible attributes are:
  * `value` - this is the name of `dataTable` item property.
  * `title` - sets the title of the column
  * `type` - sets the type of the column (used by editors/renderers), e.g. `java.lang.Integer`
  * `width` - sets column width
  * `renderer` - sets column renderer. This is EL pointing to propper `TableCellRenderer` instance
  * `rendererClass` - sets column renderer. The value is the full classname of renderer. Renderer must contain default constructor. This attribute is ignored if `renderer` is specified
  * `editor` - sets column editor. This is EL pointing to propper `TableCellEditor` instance
  * `editorClass` - sets column editor. The value is the full classname of editor. Editor must contain default constructor. This attribute is ignored if `editor` is specified


Example:

form:
```
<c:dataTable value="${model.persons}">
   <c:column value="name" title="Name" renderer="${nameRenderer}" editorClass="foo.MyCustomTableCellEditor">
</c:dataTable>
```

controller:
```
public class Form extends JPanel {

   @ModelElement("nameRenderer")
   private TableCellRenderer nameRenderer = new MyCustomTableCellRenderer();

}
```


custom editor:
```
package foo;

public class MyCustomTableCellEditor implements TableCellEditor {
   public MyCustomTableCellEditor() {} //default constructor
}
```
### combobox ###
Tag is converted to the `javax.swing.JComboBox`.

Currently it supports only one child tag: an `selectItems` tag, which holds the EL for list model. If it doesn't present,
no model is installed.

JComboBox's model is evaluated with jsr-255 binding. Source bean property must be of type `java.util.List`. Use
`org.jdesktop.observablecollections.ObservableCollections.observableList()` to create observable list, so adding/removing
elements to/from collection will add/remove elements from JComboBox model.

You can bind selected combobox element with `selectedElement` attribute. Currently, due to limitations of better beans
binding, this is readonly bindings (changing model won't update JList selection).

```
<c:combobox align='center' selectedElement="${model.runtime}">
    <c:selectItems value="${runtimeValues}"/>
</c:combobox>
```

```
public class CreateProjectForm extends JPanel {

    ...

    @ModelElement("runtimeValues")
    private List<String> runtimeValues = Arrays.asList("Apache Tomcat v5.5", "Apache Tomcat v6.0");

    @ModelElement("model")
    private Model model = new Model();

    ...
}
```


### dataTable ###
Tag is converted to the `javax.swing.JTable`.

Use this tag to iterate through list of simular items (with binding to it). Items will be displayed in table, one item per row.
Data will be retrieved with `value` attribute value. It is EL, pointing to `java.util.List` model's property.
Use `org.jdesktop.observablecollections.ObservableCollections.observableList()` to automatically update table's
model on adding/removing element to model's list.

To specify columns for a table, use `<c:column/>` tag. The `value` attribute of `column` tag must resolve item's
property to display in this column.


Example:

domain object:
```
public class Person {
    private String name;
    private String email;
    ... //getters and setters with propertyChangeSupport usage
}
```


model:
```
public class Model {
    private List<Person> persons;
    private Person selectedPerson;

    public Model() {
        persons = org.jdesktop.observablecollections.ObservableCollections.observableList(new ArrayList());
        //fill some data
        persons.add(new Person());
        persons.add(new Person());
        persons.add(new Person());
    }

    ...//getters and setters
}
```

controller:
```
public class FormPanel extends JPanel {

    @ModelAttribute("model")
    private Model model = new Model();

    ...
}
```

form:
```
<c:dataTable value="${model.persons}" selectedElement="${model.selectedPerson}">
    <c:column value="name" title="Name">
    <c:column value="email">
</c:dataTable>
```

in this example we create table with 2 columns: name and email. Both resolve corresponding properties of `Person` class.
Table will have 3 rows (as we added 3 persons in `Model` constructor). Take a look that all table's cells are binded
to model values, so changing cell's value will update model's value.


To change selection model of the table, use `selectionType` attribute. Possible values are:
  * `single` - the same as `ListSelectionModel.SINGLE_SELECTION`
  * `multiple` - the same as `ListSelectionModel.SINGLE_INTERVAL_SELECTION`
  * `custom` - the same as `ListSelectionModel.MULTIPLE_INTERVAL_SELECTION`

You can track selection with `selectedElement` attribute (`selectedElements` to track multiple selection).
The value of this attribute is EL pointing to model's property. This is read-only binding, so changing model's
property value won't change selection, but changing selection will update property's value.


Use `autoresize` attribute to specify table's column resizing strategy (JTable.setAutoResizeMode()). Possible values are:
  * `off` - the same as `JTable.AUTO_RESIZE_OFF`
  * `all` - the same as `JTable.AUTO_RESIZE_ALL_COLUMNS`
  * `last` - the same as `JTable.AUTO_RESIZE_LAST_COLUMN`
  * `next` - the same as `JTable.AUTO_RESIZE_NEXT_COLUMN`
  * `auto` - the same as `JTable.AUTO_RESIZE_SUBSEQUENT_COLUMNS`



### editorPane ###
Tag is converted to the `javax.swing.JEditorPane`. The content type for the document is set with `type` attribute:
```
<c:editorPane value="${model.selectedProject.description}" type='text/html' readonly='true' >
    <c:attribute name="preferredSize" value="0 80" type='java.awt.Dimension'/>
</c:editorPane>
```

The `value` attibute, if presents, sets the binding for the component. If it is absent, then usual text component
attributes are applied: the content of the tag is set as component text.




### formTable ###
This tag allows you to bind specific cells to model properties. It is converted to `javax.swing.JTable`.

Supported attributes:
  * `autoresize` - see below
  * `showHeader` - show/hide table's header



Use `autoresize` attribute to specify table's column resizing strategy (JTable.setAutoResizeMode()). Possible values are:
  * `off` - the same as `JTable.AUTO_RESIZE_OFF`
  * `all` - the same as `JTable.AUTO_RESIZE_ALL_COLUMNS`
  * `last` - the same as `JTable.AUTO_RESIZE_LAST_COLUMN`
  * `next` - the same as `JTable.AUTO_RESIZE_NEXT_COLUMN`
  * `auto` - the same as `JTable.AUTO_RESIZE_SUBSEQUENT_COLUMNS`

Tag supports child elements the same way, the usual `<table>` tag does. No cellspan/rowspan is supported (since JTable
doesn't support it). `th` is not supported.

The cell content must be specified with `<td>` tag. To output text, just place it inside `td`. To provide binding
with model fill `value` attribute of `td` with propper EL:
```
<c:formTable>
  <tr>
    <td>Name</td>
    <td value='${model.person.name}'></td>
  </td>
</c:formTable>
```

The `td` child tag supports attributes:
  * `width` - sets column width
  * `height` - sets row height
  * `readonly` - disables cell editing (false by default)
  * `renderer` - sets column renderer. This is EL pointing to propper `TableCellRenderer` instance
  * `editor` - sets column editor. This is EL pointing to propper `TableCellEditor` instance

Example:
```
<c:formTable autoresize='last' showHeader='false'>

  <tr>
      <td width="150" readonly="true">Name</td>
      <td width="50" value='${person.name}'></td>

      <td width="150" readonly="true">Last name</td>
      <td width="50" value='${person.lastName}'></td>
  </tr>
  <tr>
      <td readonly="true">Age</td>
      <td value="${person.age}" editor="${spinnerEditor}"></td>

      <td readonly="true">Active</td>
      <td value="${person.active}" editor="${booleanEditor}" renderer="${booleanRenderer}"></td>
  </tr>
  <tr>
      <td readonly="true">Color</td>
      <td height="50" value="${person.color}" renderer='${colorRenderer}' editor="${colorEditor}"></td>

      <td readonly='true'>Comment</td>
      <td editor="${textAreaEditor}"></td>
  </tr>

</c:formTable>

```

controller:
```
public class FormTableForm extends JPanel {

    private DomModel domModel;

    @ModelElement("person")
    private Person person;

    @ModelElement("colorEditor")
    private ColorTableCellEditor colorEditor = new ColorTableCellEditor();

    @ModelElement("spinnerEditor")
    private SpinnerEditor spinnerEditor = new SpinnerEditor();

    @ModelElement("booleanEditor")
    private BooleanEditor checkboxEditor = new BooleanEditor();

    @ModelElement("colorRenderer")
    private ColorTableCellRenderer colorRenderer = new ColorTableCellRenderer();

    @ModelElement("booleanRenderer")
    private BooleanRenderer checkboxCellRenderer = new BooleanRenderer();

    @ModelElement("textAreaEditor")
    private TextAreaEditor textAreaEditor = new TextAreaEditor();

    ...
}
```





### glue ###
Tag is converted to the component, that is returned from
```
javax.swing.Box.createHorizontalGlue();
```
for  `type="horizontal"`, or
```
javax.swing.Box.createVerticalGlue();
```
for `type="vertical"`. Default `type` value is `vertical`.

Used within tags with `BoxLayout` layout

Example:
```
<c:glue type="horizontal"/>
```


### list ###
Tag is converted to the `javax.swing.JList`.

Currently it supports only one child tag: an `selectItems` tag, which holds the EL for list model. If it doesn't present,
no model is installed.

List's model is evaluated with jsr-255 binding. Source bean property must be of type `java.util.List`. Use
`org.jdesktop.observablecollections.ObservableCollections.observableList()` to create observable list, so adding/removing
elements to/from collection will add/remove elements from JList model.

You can bind selected list element with `selectedElement` attribute. `selectedElements` can be used to bind
many selected elements. Currently, due to limitations of better beans binding, these are readonly bindings (changing
model won't update JList selection).

You can set renderer for the list with `renderer` attribute. This must be EL, pointing to the `javax.swing.ListCellRenderer`
instance.


```
//controller
public class MyPanel {

   @ModelElement("model")
   private MyPanelModel model;

    @ModelElement("customRenderer")
    private CustomRenderer projectTypeRenderer = new CustomRenderer();

   ...

   public void foo() {
      items.add("Line 1");
      items.add("Line 2");
   }


}


//model
public class MyPanelModel {
   private PropertyChangeSupport pcs = new PropertyChangeSupport(this);
   private List items = ObservableCollections.observableList(new ArrayList());
   private String selected;
   //getters and setters with PropertyChangeSupport firing

   ...

}

//renderer
public class CustomRenderer extends DefaultListCellRenderer implements ListCellRenderer {
   ...
}
```

```
<c:editorPane selectedElement="${model.selected}" renderer="${customRenderer}">
   <c:selectItems value="${model.items}"/>
</c:editorPane>
<input type='button' onclick='foo'/>
```



### selectItems ###
Special tag to specify model items for `combobox`, `list`, `dataTable` etc tags. It supports one attribute:
`value`. It is EL pointing to propper model property, which type must be `java.util.List`.

```
<c:combobox selectedElement="${model.runtime}">
    <c:selectItems value="${runtimeValues}"/>
</c:combobox>
```


### scroll ###
Tag is converted to `javax.swing.ScrollPane`. The first child tag component will be placed in the viewport. If there are
more than 1 child tags, warning will be printed to the log.

Example:
```
<c:scroll>
    <object classid="javax.swing.JTable"/>
</c:scroll>
```


### spinner ###
Tag is converted to `javax.swing.JSpinner`
Tag supports attributes:
  * value - en EL expression, describing a property of a model element, the value of the component is binded to.

Example:
```
<c:spinner />
```


### split ###
Tag is converted to `JSplitPane`.

Tag supports attributes:
  * `orientation` - values: horizontal, vertical
  * `divider-size` - as in JSplitPane
  * `divider-position` - as in JSplitPane. Value is in pixels (100) or in percents (50%)


Tag can has no more than 2 child components. Childs are placed according to their `align` preperty:
  * `left` - component goes to left panel
  * `top` - component goes to top panel
  * `right` - component goes to right panel
  * `bottom` - component goes to bottom panel

If first child doesn't have `align` property, then first child goes to left panel, second - to right panel.

Example:
```
<c:split orientation="horizontal" style="divider-size: 2;divider-position: 200;">

    <c:scroll>
        <textarea>Left</textarea>
    </c:scroll>

    <c:scroll>
        <textarea>Right</textarea>
    </c:scroll>

</c:split>
```


### strut ###
Tag is converted to the component, that is returned from
```
javax.swing.Box.createHorizontalStrut(w)
```
for `type="horizontal"`, or
```
javax.swing.Box.createVerticalStrut(h)
```
for `type="vertical"`. `type` has default value `vertical`.

Used within tags with `BoxLayout` layout

`w` and `h` - are values of attributes `width` и `height` accordingly.

Example:
```
<c:strut type="horizontal" style="width: 12"/>
<c:strut type="vertical" height="12"/>
```

### tabs ###
Tag is converted to `JTabbedPane`.

Tab is created for every direct child component, `title` attribute value is used as tab's title.

Tag supports attributes:

  * `tab-position` - tabs position. Values: `top`, `bottom`, `left`, `right`.


Example:
```
<c:tabs>
    <div title="Text">
        <p>Tab 1</p>
    </div>

    <div title="JEditorPane" style="display:border">
        <c:scroll>
            <object classid="javax.swing.JEditorPane"/>
        </c:scroll>

    </div>
</c:tabs>
```



### tree ###
The tag is converted to `javax.swing.JTree` component.

Supported attributes:
  * `value` - EL, pointing to `javax.swing.tree.TreeModel` instance which will be used as tree model
  * `showRoot` - show tree root or not. Values: `true` or `false`
  * `showRootHandles` - show tree root handles or not. Values: `true` or `false`

The tag supports events:
  * `onchange` - the name of the controller's method to invoke on selection change. The method must take no arguments or take 1 argument of type `javax.swing.event.TreeSelectionEvent`

Example:
```
<c:scroll>
    <c:tree value="${categoriesTreeModel}" showRoot="false" onchange="onCategoryChange" showRootHandles="true"/>
</c:scroll>
```










## `http://swinghtmltemplate.googlecode.com/ui` namespace ##

### composition ###
This tag allows you to compose dom model with template, almost the same way as in facelets.

The template is assigned with `template` attribute. The value of the `template` must be in
`getClass().getResourceAsStream()` format.

This tag must contain only `<define>` children. Each child defines an named snippet to be inserted into
template.


When composition is used, everything outside the `composition` tag is ignored.

Example:
```
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN"
  "http://www.w3.org/TR/html4/loose.dtd">
<html xmlns:c='http://www.oracle.com/swing'
      xmlns:ui='http://swinghtmltemplate.googlecode.com/ui'>
<head>
    <title></title>
</head>
<body>

    <ui:composition template='ru/swing/html/tags/ui/CompositionTemplate.html'>

        <ui:define name='content'>
            <c:scroll>
                <textarea>
            </c:scroll>
        </ui:define>

        <ui:define name='control'>
            <div>
                <input type="button" text="OK">
                <input type="button" text="Cancel">
            </div>
        </ui:define>

    </ui:composition>


</body>
</html>

```


### define ###
This tag defines an snippet to be inserted into template using `<composition>` tag.
The required attribute is `name` which defines a name of the snippet. This name is used in `<insert>`
tag when template is built.



### include ###
This tag allows you to include another document (target) content into your document (source).

The address of the target is assigned with `src` attribute. Target document must contain `<body>` tag.
All direct children of target's body tag will be inserted into the parent of the `<include>` tag in the
source document. The `<include>` tag will be removed from it's parent.

Example:
source:
```
<html>
<body>
    <div>
        <include src='ru/example/target.html'/>
    </div>
</body>
</html>
```

target (`/src/ru/example/target.html`):
```
<html>
<body>
    <input type='button'/>
</body>
</html>
```

result:
```
<html>
<body>
    <div>
        <input type='button'/>
    </div>
</body>
</html>
```


The value of the `src` attribute is the address of the target html file in `getClass().getResourceAsStream()` format.



### insert ###
This tag inserts named snippet (defined with `<define>` tag) into a template. The name of the snippet
is set with `name` attrubute.




# Layout managers #
Layout manager for a component is assigned using `display` attribute.

## absolute ##
Means null layout.

Child components are placed according to `getBounds()` value. Bounds can be specified with `align`
attribute in the format of `java.awt.Rectangle`, or using `<attribute>` child tag. `align` value
has higher priority than `<attribute>`.

```
<div style='display:absolute'>
   <p align='10 10 10 10'>block 1</p>
   <p>
      <c:attribute name='bounds' value='30 10 10 10' type='java.awt.Rectangle'/>
      block 2
   </p>
</div>
```

## border ##
Means `java.awt.BorderLayout`.

Child components are placed according to `align` property:
  * `align="top"` - north
  * `align="bottom"` - south
  * `align="left"` - west
  * `align="right"` - east
  * `align="center"` - center
Component is placed in the center by default.

`margin` value, written in `margin:hgap vgap` format, is used as horizontal and vertical gaps.

Example:
```
<div style="display: border; margin: 6 0">
   <input type="button" text="ok"/>
   <input type="button" text="cancel"/>
</div>
```


## box ##
Means `javax.swing.BoxLayout`.

Direction of child components is specified with `x-boxlayout-direction`:
  * `horizontal` - components are placed horizontally
  * `vertical` - components are placed vertically

You can use `<strut/>`  и `<glue/>` to put spaces between components.

Example:
```
<div style="display: box; x-boxlayout-direction: horizontal;border: empty 0 0 0 6">
   <c:glue type="horizontal"/>
   <input type="button" text="OK" id="ok" icon="/img/accept.png" />
   <strut type="horizontal" style="width: 6"/>
   <input type="button" text="Cancel" id="cancel" icon="/img/cancel.png"/>
</div>
```

## flow ##
Means `java.awt.FlowLayout`.

Children are aligned according to `text-align`:
  * `center` - components are aligned in center
  * `left` - components are aligned to the left
  * `center` - components are aligned to the right

`margin` value, written in `margin:hgap vgap` format, is used as horizontal and vertical gaps.


Example:
```
<div style="display: flow; text-align: right; margin: 10 0">
    <input id="ok" type="button" text="OK"/>
    <input type="button" text="Cancel"/>
</div>
```

## mig ##
Means [net.miginfocom.swing.MigLayout](http://www.miglayout.com/).

Layout constraints are set in `x-miglayout-constraints` attribute.

Columns constraints are set in  `x-miglayout-column-constraints` attribute.

Rows constraints are set in  `x-miglayout-row-constraints` attribute.

Example:
```
<form id="loginForm" style="x-miglayout-column-constraints: [right]related[grow,fill];">
   <p>Login:</p>
   <input id="login" type="text" align="wrap"/>
   <p>Password:</p>
   <input id="password" type="password" align="wrap" />
</form>
```

Tag `<form/>` uses this layout by default.

## table ##
Means [TableLayout](https://tablelayout.dev.java.net).

Column sizes are set with `x-tablelayout-column-sizes` attribute.

Row sizes are set with `x-tablelayout-row-sizes` attribute.

Examples:
```
<table style="x-tablelayout-row-sizes: preferred preferred preferred preferred; x-tablelayout-column-sizes: 100 fill; border: compound (etched) (empty 12 12 12 12)">

    <tr>
        <td style="text-align: right;"><p>Login:</p></td>
        <td><input type="text">qqq</input></td>
    </tr>
    <tr>
        <td style="text-align: right;"><p>Password:</p></td>
        <td>
            <input type="text">***</input>
        </td>
    </tr>
    <tr>
        <td style="text-align: right;"><p>Remember:</p></td>
        <td>
            <input type="checkbox"/>
        </td>
    </tr>
    <tr>
        <td style="text-align: right;vertical-align:top"><p>Login as:</p></td>
        <td>
            <table style="x-tablelayout-row-sizes: preferred preferred; x-tablelayout-column-sizes: preferred fill">
                <tr>
                    <td><input type='radio'/></td>
                    <td><p>Manager</p></td>
                </tr>
                <tr>
                    <td><input type='radio'/></td>
                    <td><p>Admin</p></td>
                </tr>
            </table>
        </td>
    </tr>

</table>

<div style="display: table; x-tablelayout-row-sizes: preferred preferred; x-tablelayout-column-sizes: 100 fill;">
   <p align="0 0 c c">1</p>
   <p align="0 1 c c">2</p>
   <p align="1 0 c c">3</p>
   <p align="1 1 c c">4</p>
</div>
```

Tag `<table/>` uses this layout.



# Events #
Different event listeners can be attached to components. Supported event types are:
  * `onclick` - it can be assigned to `javax.swing.AbstractButton` descendants. An corresponding method in controller is invoked on click.
  * `onchange` - it can be assigned to `javax.swing.text.JTextComponent` descendants. An corresponding method in controller is invoked on document change.


## onclick ##
Make sure an controller is binded to a model.
A method, whose name is equal to `onclick` value, is searched in controller. You can make 2 type of methods:
  * no-params (is searched first)
  * 1-param (java.awt.event.ActionEvent) (is searched if there's no no-param method)
```
<html>
   <head></head>
   <body>
      <input type='button' onclick='foo' text='Click me!'/>
   </body>
</html>
```
```
public class Foo {
   public Foo() {
      try {
          model = Binder.bind(this, true);
      } catch (JDOMException e) {
          e.printStackTrace();
      } catch (IOException e) {
          e.printStackTrace();
      }
   }

   public void foo() {
      System.out.println("Foo is clicked");
   }
}
```

## onchange ##
Make sure an controller is binded to a model.
A method, whose name is equal to `onclick` value, is searched in controller. You can make 2 type of methods:
  * no-params (is searched first)
  * 1-param (javax.swing.event.DocumentEvent) (is searched if there's no no-param method)
```
<html>
   <head></head>
   <body>
      <input id='txt' type='text' onchange='foo'/>
   </body>
</html>
```
```
public class Foo {
   public Foo() {
      try {
          model = Binder.bind(this, true);
      } catch (JDOMException e) {
          e.printStackTrace();
      } catch (IOException e) {
          e.printStackTrace();
      }
   }

   public void foo() {
      System.out.println("Foo text has changed");
   }
}
```

# Styles #
Tag attribute can be set separately of the tag by using `<style/>` within `<head/>`, as with ordinary
css styles in html.

Style is set in css format. Almost all CSS2 selectors supported, examples:
  * `*`
  * tag\_name
  * tag\_name[attr='value']
  * tag\_name[attr=value]
  * tag\_name[attr|=value]
  * tag\_name[attr~=value]
  * **[attr~=value]
  * `*.foo.red`
  * .class\_name.class2
  * #identifier
Pseudo-elements and pseudo-classes are not supported (as there is no need in them).
Example:
```
<style>
div {
   display: border;
}
p.error {
   color: red;
}
div.red.top {
   vertical-align: top;
   color: red;
}

#results {
   color: red;
}
</style>
```**

There is no difference between tag attributes and css styles.
So these are identical:
```
<html>
<head>
   <style>
   .ok {
      text: ok;
   }
   </style>
</head>
<body>
   <input type='button' class='ok'/>
</body>
</html>
```
```
<html>
<body>
   <input type='button' style='text:ok'/>
</body>
</html>
```
```
<html>
<body>
   <input type='button' text='ok'/>
</body>
</html>
```
`style` attribute values overrides css styles in `style` tag.


# Borders #
You can set component's border with `border` attribute.


Border format:
```
border_type border_params
```

Supported border types are:

## compound ##
Compount border. 2 parameters: outer border and inner border.
`compound (outer) (inner)`
Example:
```
compound (empty 12 12 12 12) (compound (line black) (empty 12 12 12 12))
```


## empty ##
Empty border. No parameters or 4 integers (insets): `top left bottom right`


```
border: empty;
border: empty 12 12 12 12;
```


## etched ##
Etched border. Supported formats:
```
etched
etched type
etched highlight_color shadow_color
etched type highlight_color shadow_color
```
where
  * `type` - border type (`lowered`, `raised`)
  * `highlight_color` - highlight color in format of `ColorFactory.getColor(String)`
  * `shadow_color` - shadow color in format of `ColorFactory.getColor(String)`

Examples:
```
border: etched;
border: etched lowered;
border: etched white black;
border: etched lowered white black;
```


## line ##
Line border. Supported formats:
```
line color
line color thick
```
where
  * `color` - line color in format of `ColorFactory.getColor(String)`
  * `thick` - line width
Examples:
```
border: line black;
border: line red 2;
```



## titled ##
Titled border. Supported formats:
```
titled "caption"
titled "caption" (inner)
titled "caption" (inner) justification position
titled "caption" (inner) justification position (font) color
```
where
  * `caption` - border caption
  * `inner` - inner border
  * `justification` - horizontal align (`left`, `right`, `center`, `leading`, `trailing`)
  * `position` - vertical align (`top`, `above-top`, `below-top`, `bottom`, `above-bottom`, `below-bottom`)
  * `font` - caption's font in format of `java.awt.Font.decode(String)`
  * `color` - caption's color in format of `ColorFactory.getColor(String)`
Examples:
```
titled "Foo"
titled "Foo" (empty 12 12 12 12)
titled "Foo" (empty 12 12 12 12) left bottom
titled "Foo" (empty 12 12 12 12) left bottom (Arial) red
```



## Examples ##
```
<body style="border: compound (empty 12 12 12 12) (compound (etched) (empty 12 12 12 12))">
   <div style="border: line black"><p>Hello</p></div>
</body>
```