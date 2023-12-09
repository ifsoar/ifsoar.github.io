# FXML简介（Introduction to FXML）
最后更新：2012 年 6 月 21 日

## 内容

- [[#概述]]
- [[#元素 Elements]]
	- [[#类实例元素]]
		- [[#实例声明标签]]
		- [[#<fx:include> 包含标签]]
		- [[#<fx:constant> 常量标签]]
		- [[#<fx:reference> 引用标签]]
		- [[#<fx:copy> 复制标签]]
		- [[#<fx:root> 根标签]]
	- [[#属性元素标签]]
		- [[#属性设置器标签]]
		- [[#只读List属性标签]]
		- [[#只读Map属性标签]]
		- [[#默认属性]]
	- [[#静态属性标签]]
	- [[#<fx:define>定义块]]
- [[#元素的属性]]
	- [[#类实例的属性]]
		- [[#定位解析]]
		- [[#资源解析]]
		- [[#变量解析]]
		- [[#转义]]
	- [[#元素的静态属性]]
	- [[#事件处理器属性]]
		- [[#脚本事件处理器]]
		- [[#控制器方法事件处理器]]
- [[#脚本]]
- [[#控制器]]
	- [[#@FXML注解]]
	- [[#嵌套控制器]]
- [[#FXML加载器]]
	- [[#自定义组件]]

## 概述
FXML 是一种可编写脚本的、基于 XML 的标记语言，用于构建 Java 对象图。它给通过代码构建这类图提供了一种方便的替代方法，并且非常适合用来定义 JavaFX 应用程序的用户界面，因为 XML 文档的层次结构与 JavaFX 场景图的结构非常相似。

本文档介绍了 FXML 标记语言并解释了如何使用它来简化 JavaFX 应用程序的开发。
> 翻译注：  
> 在xml结构中，左右尖括号成对出现包起来的内容称为标签，文中称为元素；
> 标签可以有自己的属性，属性可以是‘key’=‘value’的形式，也可以只有属性名

## 元素 Elements

在 FXML 中，XML 元素用来表示以下几种内容：

-   类实例
-   类实例的属性
-   静态属性
-   一个定义块
-   一段脚本代码

类实例、实例属性、静态属性和定义块将在下面的本节中讨论。脚本将在后面的部分中讨论。

### 类实例元素
可以通过多种方式在 FXML 中构造类实例。最常见的是通过实例声明的方式，也就是通过类名创建一个类的新实例。创建类实例的其他方法包括引用现有值、复制现有值以及包括外部 FXML 文件。下面将更详细地讨论每一个。

#### 实例声明标签
如果一个元素的标签名称以大写字母开头（并且它不是一个静态属性设置器，稍后提到），则被认为是一个实例声明。当 FXML 加载器（稍后提到）遇到这样的元素时，它会创建该类的一个实例。

与在 Java 中一样，类名可以是完全限定的（包括包名），也可以使用“import”处理指令 (processing instruction = PI) 导入。  

例如，以下 PI指令将 javafx.scene.control.Label 类导入到当前 FXML 文档的命名空间中：
```xml
	<?import javafx.scene.control.Label?>
```

 以下指令将 javafx.scene.control 包中的所有类导入当前命名空间：
```xml
	 <?import javafx.scene.control.*?>
```

任何遵循 JavaBean 构造函数和属性命名约定的类都可以使用 FXML 轻松实例化和配置。

以下是创建javafx.scene.control.Label实例并将其“text”属性设置为“Hello, World!”的简单但完整的示例：
```xml
<?import javafx.scene.control.Label?>
<Label text="Hello, World!"/>
```

请注意，此示例中Label的“文本”属性是使用 XML 属性设置的。也可以使用嵌套的属性元素来设置属性。本节稍后将更详细地讨论属性元素。Property 属性将在后面的部分中讨论。

不符合 Bean 约定的类也可以使用称为“构建器”的对象在 FXML 中构造。稍后将更详细地讨论构建器。

##### 字典 Maps
在内部，FXML 加载器使用com.sun.javafx.fxml.BeanAdapter的实例来包装实例化对象并调用其设置方法。这个（当前）私有类实现了java.util.Map接口并允许调用者获取和设置 Bean 属性值作为键/值对。

如果一个元素代表一个已经实现了Map 的类型（例如java.util.HashMap），它不会被包装，而是直接调用它的get()和put()方法。例如，以下 FXML 创建一个HashMap实例并将其“foo”和“bar”值分别设置为“123”和“456”：

```xml
<HashMap foo="123" bar="456"/>
```

##### fx:value 属性
fx :value属性可用于初始化没有默认构造函数但提供静态valueOf(String)方法的类型的实例。例如，java.lang.String以及每个原始包装器类型都定义了一个valueOf()方法，并且可以在 FXML 中如下使用：

```xml
<String fx:value="Hello, World!"/>
<Double fx:value="1.0"/>
<Boolean fx:value="false"/>
```
也可以通过这种方式构造定义了静态的valueOf(String)方法的自定义类。

##### fx:factory 属性

fx:factory属性是创建其类没有默认构造函数的对象的另一种方法。属性的值是用于生成类实例的静态、无参数工厂方法的名称。例如，以下标记创建了一个可观察数组列表的实例，其中填充了三个字符串值：
```xml
<FXCollections fx:factory="observableArrayList"> 
	<String fx:value="A"/> 
	<String fx:value="B"/> 
	<String fx:value="C"/> 
</FXCollections>
```

##### Builder

创建不符合 Bean 约定的类实例（例如那些表示不可变值的实例）的第三种方法是“Builder”。Builder设计模式将对象创建委托给负责制造不可变类型实例的可变助手类（称为“Builder”）。

FXML 中的Builder支持由两个接口提供。javafx.util.Builder接口定义了一个名为build()的方法，它负责构建实际对象：
```java
public interface Builder<T> { 
	public T build(); 
}
```

javafx.util.BuilderFactory负责生成能够实例化给定类型的构建器：
```java
public interface BuilderFactory { 
	public Builder<?> getBuilder(Class<?> type); 
}
```

javafx.fxml包中提供了默认构建器工厂JavaFXBuilderFactory。该工厂能够创建和配置大多数不可变的 JavaFX 类型。例如，以下代码使用默认构建器创建不可变类 javafx.scene.paint.Color的实例：
```xml
<Color red="1.0" green="0.0" blue="0.0"/>
```

请注意，与在处理元素的开始标记时构造的 Bean 类型不同，由Builder构造的对象在到达元素的结束标记之前不会被实例化。这是因为在元素被完全处理之前，所有必需的参数可能都不可用。例如，上例中的 Color 对象也可以写成：
```xml
<Color>
    <red>1.0</red>
    <green>0.0</green>
    <blue>0.0</blue>
</Color>
```

在已知所有三个颜色分量之前，无法完全构造Color实例。

在处理将由Builder构造的对象的标记时，Builder实例被视为值对象 - 如果Builder实现了Map接口，则使用put()方法设置Builder的属性值。否则，Builder被包装在 BeanAdapter 中，并且假定其属性通过标准 Bean setter 公开。

#### <fx:include> 包含标签

<fx:include>标签从另一个文件中定义的 FXML 标记创建一个对象。它的用法如下：
```xml
<fx:include source="filename"/>
```

其中filename是要包含的 FXML 文件的名称。以前导斜杠字符【/】开头的值被视为相对于Classpath路径。没有前导斜杠的值被认为是相对于当前文档的路径。

例如，给定以下标记：
```xml
<?import javafx.scene.control.*?> 
<?import javafx.scene.layout.*?> 
<VBox xmlns:fx="http://javafx.com/fxml"> 
	<children> 
		<fx:include source ="my_button.fxml"/> 
	</children> 
</VBox>
```


如果my_button.fxml包含以下内容：
```xml
<?import javafx.scene.control.*?>
<Button text="My Button"/>
```

生成的场景图将包含一个VBox作为根对象，并包含一个Button作为子节点。

请注意“fx”命名空间前缀的使用。这是一个保留的前缀，它定义了一些用于 FXML 源文件内部处理的元素和属性。它通常在 FXML 文档的根元素上声明。“fx”命名空间提供的其他功能将在以下各节中介绍。

<fx:include>还支持用于指定应该用于本地化所包含内容的资源包名称的属性，以及用于对源文件进行编码的字符集。资源解析将在后面的部分讨论。

#### <fx:constant> 常量标签

<fx:constant>元素创建对类常量的引用。例如，以下标记将Button实例的“minWidth”属性的值设置为java.lang.Double类定义的NEGATIVE_INFINITY常量的值：
```xml
<Button>
    <minHeight>
	    <Double fx:constant="NEGATIVE_INFINITY"/>
	</minHeight>
</Button>
```

#### <fx:reference> 引用标签

<fx:reference>元素创建对现有元素的新引用。无论此标记出现在哪里，它都将有效地替换为命名元素的值。它与fx:id属性或脚本变量一起使用，这两者将在后面的部分中进行更详细的讨论。<fx:reference>元素的“source”属性指定新元素将引用的对象的名称。

例如，以下标记将先前定义的名为“myImage”的Image实例分配给ImageView控件的“image”属性：
```xml
<ImageView>
    <image>
        <fx:reference source="myImage"/>
    </image>
</ImageView>
```

请注意，由于也可以使用属性变量解析运算符取消引用变量（稍后在属性部分讨论），fx:reference通常仅在必须将引用值指定为元素时使用，例如添加参考集合：
```xml
<ArrayList>
    <fx:reference source="element1"/>
    <fx:reference source="element2"/>
    <fx:reference source="element3"/>
</ArrayList>
```

对于大多数其他情况，使用属性更简单、更简洁。

#### <fx:copy> 复制标签

<fx:copy>元素创建现有元素的副本。与<fx:reference>一样，它与 fx:id 属性或脚本变量一起使用。元素的“source”属性指定将被复制的对象的名称。源类型必须定义一个复制构造函数，用于从源值构造副本。

目前，没有任何 JavaFX 平台类提供这样的复制构造函数，因此此元素主要供应用程序开发人员使用。这可能会在未来的版本中改变。

#### <fx:root> 根标签

<fx:root>元素创建对先前定义的根元素的引用。它仅作为 FXML 文档的根节点有效。<fx:root>主要用于创建由 FXML 标记支持的自定义控件。FXMLLoader部分对此进行了更详细的讨论。

### 属性元素标签

标签名称以小写字母开头的元素称为属性元素标签。属性元素可以表示以下几种情况：

-   属性设置器
-   只读List属性
-   只读Map属性

#### 属性设置器标签

如果元素表示属性设置器，则元素的内容（必须是文本节点或嵌套类实例元素）作为值传递给属性的设置器。

例如，以下 FXML 创建Label类的一个实例并将标签的“text”属性的值设置为“Hello, World!”：
```xml
<?import javafx.scene.control.Label?>
<Label>
    <text>Hello, World!</text>
</Label>
```

这会产生与前面示例相同的结果，该示例使用属性设置器来设置“text”属性：
```xml
<?import javafx.scene.control.Label?>
<Label text="Hello, World!"/>
```

当属性值是一个复杂类型，无法使用简单的基于字符串的属性值表示时，或者当值的字符长度太长以至于将其指定为属性会产生负面影响时，通常使用属性元素有更好的可读性。

##### 类型强制

FXML 使用“类型转换”根据需要将属性值转换为适当的类型。类型强制是必需的，因为 XML 支持的唯一数据类型是元素、文本和属性（其值也是文本）。然而，Java 支持许多不同的数据类型，包括内置的原始值类型以及各种引用类型。

FXML 加载器使用BeanAdapter的coerce()方法来执行任何所需的类型转换。此方法能够执行基本的原始类型转换，例如String到boolean、int到double等，也可以将String转换为Class或String到Enum。我们可以通过在目标类型上定义静态valueOf()方法来实现其他转换。

#### 只读List属性标签

只读List属性是一个 Bean 属性，其 getter 返回java.util.List的实例并且没有相应的 setter 方法。只读列表元素的内容在处理时会自动添加到列表中。

例如，javafx.scene.Group的“children”属性是一个只读列表属性，表示该组的子节点：
```xml
<?import javafx.scene.*?> 
<?import javafx.scene.shape.*?> 
<Group xmlns:fx="http://javafx.com/fxml"> 
	<children> 
		<Rectangle 
			fx:id="rectangle" x="10" y="10" width="320" height="240" fill="#ff0000"/>
		...
		</children>
</Group>
```

当 children 元素的每个子元素被读取时，子元素会被添加到Group的getChildren()方法返回的列表中。

#### 只读Map属性标签

只读Map属性是一个 bean 属性，其 getter 返回java.util.Map的实例并且没有相应的 setter 方法。处理结束标记时，只读map元素的属性将应用于Map。

javafx.scene.Node的“properties”属性是只读Map属性的示例。

以下标记分别将Label实例的“foo”和“bar”属性设置为“123”和“456”：
```xml
<?import javafx.scene.control.*?>
<Button>
    <properties foo="123" bar="456"/>
</Button>
```

请注意，类型既不是List也不是Map的只读属性将被视为只读Map。getter 方法的返回值将被包装在 BeanAdapter 中，并且可以像任何其他只读Map一样使用。

#### 默认属性

类可以使用javafx.beans包中定义的@DefaultProperty注解来定义“默认属性” 。如果存在，表示默认属性的子元素可以从标记中省略。

例如，由于javafx.scene.layout.Pane （ javafx.scene.layout.VBox的超类）定义了“children”的默认属性，因此不需要 children 元素；加载器会自动将VBox的子元素添加到容器的“children”集合中：
```xml
<?import javafx.scene.*?>
<?import javafx.scene.shape.*?>
<VBox xmlns:fx="http://javafx.com/fxml">
    <Button text="Click Me!"/>
    ...
</VBox>
```

请注意，默认属性不限于集合。如果元素的默认属性引用标量值，则该元素的任何子元素都将设置为该属性的值。

例如，由于javafx.scene.control.ScrollPane定义了默认属性“content”，因此可以按如下方式指定 包含TextArea作为其内容的滚动窗格：
```xml
<ScrollPane>
    <TextArea text="Once upon a time..."/>
</ScrollPane>
```

利用默认属性可以显著降低 FXML 标记的冗长程度。

### 静态属性标签

一个元素也可以表示一个静态属性（有时称为“附加属性”）。静态属性标签是仅在特定上下文中才有意义的属性。它们不是应用它们的类所固有的，而是由另一个类（通常是控件的父容器）定义的。

静态属性标签以定义它们的类的名称为前缀。例如，以下 FXML 调用GridPane的“rowIndex”和“columnIndex”属性的静态设置器：
```xml
<GridPane>
    <children>
        <Label text="My Label">
            <GridPane.rowIndex>0</GridPane.rowIndex>
       <GridPane.columnIndex>0</GridPane.columnIndex>
        </Label>
    </children>
</TabPane>
```

这大致翻译成 Java 中的以下内容：
```java
GridPane gridPane = new GridPane();

Label label = new Label();
label.setText("My Label");

GridPane.setRowIndex(label, 0);
GridPane.setColumnIndex(label, 0);

gridPane.getChildren().add(label);
```

对GridPane#setRowIndex()和GridPane#setColumnIndex() 的调用将索引值“附加”到Label实例。GridPane然后在布局期间使用这些来适当地安排其子项。其他容器，包括AnchorPane、BorderPane和StackPane，定义了类似的属性。

与实例属性一样，当一个属性值不能有效表示需要的信息时，通常使用静态属性标签表示。否则，下文提到的静态属性（在后面的部分中讨论）通常会产生更简洁和可读的标记。

### <fx:define>定义块

<fx:define>元素用于创建存在于对象层次结构之外但可能需要在别处引用的对象。

例如，在使用单选按钮时，通常会定义一个ToggleGroup来管理按钮的选择状态。这个Group不是场景图本身的一部分，因此不应添加到按钮的父级。定义块可用于创建按钮组，而不会干扰文档的整体结构：
```xml
<VBox>
    <fx:define>
        <ToggleGroup fx:id="myToggleGroup"/>
    </fx:define>
    <children>
        <RadioButton text="A" toggleGroup="$myToggleGroup"/>
        <RadioButton text="B" toggleGroup="$myToggleGroup"/>
        <RadioButton text="C" toggleGroup="$myToggleGroup"/>
    </children>
</VBox>
```
define 块中的元素通常会分配一个 ID，以后可以使用该 ID 来引用元素的值。ID 将在后面的部分中进行更详细的讨论。

## 元素的属性

FXML 中的属性可以表示以下之一：

-   类实例的属性
-   “静态”属性
-   事件处理器

以下各节将更详细地讨论每一个。

### 类实例的属性

与属性元素一样，属性也可用于配置类实例的属性。例如，以下标记创建了一个文本为“Click Me!”的Button ：
```xml
<?import javafx.scene.control.*?> 
<Button text="Click Me!"/>
```

与属性元素一样，元素的属性支持类型强制。处理以下标记时，“x”、“y”、“width”和“height”值将转换为双精度值，“fill”值将转换为Color：
```xml
<Rectangle fx:id="rectangle" x="10" y="10" width="320" height="240" fill= 
    "#ff0000"/>
```

与属性元素不同，元素的属性在到达各自元素的结束标记之前不会应用。这样做主要是为了方便属性值依赖于某些信息的情况，这些信息在元素的内容被完全处理之后才可用（例如，TabPane 控件的选定索引，直到所有选项卡都已添加）。

FXML 中元素的属性和属性元素之间的另一个主要区别是元素属性支持许多扩展其功能的“解析运算符”。支持以下运算符，并在下面进行更详细的讨论：

-   资源位置解析
-   资源解析
-   变量解析

#### 定位解析

作为字符串，XML 属性本身不能表示类型化的资源位置信息，例如 URL。但是，通常需要在标记中指定此类资源位置；例如，图像资源的来源。位置解析运算符（由属性值的“@”前缀表示）用于指定应将属性值视为相对于当前文件的位置，而不是简单的字符串。

例如，以下标记创建一个 ImageView 并使用来自my_image.png的图像数据填充它，假设该图像位于相对于当前 FXML 文件的路径中：
```xml
<ImageView>
    <image>
        <Image url="@my_image.png"/>
    </image>
</ImageView>
```

由于Image是一个不可变对象，因此需要一个构建器来构建它。或者，如果Image定义一个valueOf(URL)工厂方法，则可以按如下方式填充图像视图：
```xml
<ImageView image="@my_image.png"/>
```

“image”属性的值将由 FXML 加载器转换为 URL，然后使用valueOf()方法强制转换为图像。

#### 资源解析

在 FXML 中，出于本地化目的，可以在加载时执行资源替换。当提供java.util.ResourceBundle的实例时，FXML 加载器将用特定于区域设置的值替换资源名称的实例。资源名称由“%”前缀标识，如下所示：
```xml
<Label text="%myText"/>
```

如果给加载器一个资源包，定义如下：
```
myText = This is the text!
```

FXML 加载器的输出将是一个包含文本“This is the text!”的Label实例。

#### 变量解析

FXML 文档定义了一个变量名称空间，其中可以唯一标识命名元素和脚本变量。变量解析运算符允许调用者在调用相应的 setter 方法之前用命名对象的实例替换属性值。变量引用由“$”前缀标识，如下所示：
```xml
<fx:define>
    <ToggleGroup fx:id="myToggleGroup"/>
</fx:define>
...
<RadioButton text="A" toggleGroup="$myToggleGroup"/>
<RadioButton text="B" toggleGroup="$myToggleGroup"/>
<RadioButton text="C" toggleGroup="$myToggleGroup"/>
```

将fx:id值分配给元素会在文档的命名空间中创建一个变量，稍后可以通过变量取消引用属性（例如上面显示的“toggleGroup”属性）或在稍后部分讨论的脚本代码中引用该变量。此外，如果对象的类型定义了“id”属性，该值也将传递给对象的setId()方法。

#### 转义

如果属性的值以资源解析前缀之一开头，则可以通过在其前面加上前导反斜杠（“\”）字符来转义该字符。例如，以下标记创建一个Label实例，其文本显示为“$10.00”：
```xml
<Label text="\$10.00"/>
```

#### 表达式绑定

如上所示的属性变量在加载时解析一次。以后对变量值的更新不会自动反映在分配了该值的任何属性中。在许多情况下，这就足够了；但是，将属性值“绑定”到变量或表达式通常很方便，这样对变量的更改会自动传播到目标属性。表达式绑定可用于此目的。

表达式绑定也以变量解析运算符开头，但后跟一组包裹表达式值的花括号。例如，以下标记将文本输入的“text”属性的值绑定到Label实例的“text”属性：
```xml
<TextField fx:id="textField"/>
<Label text="${textField.text}"/>
```

当用户输入文本时，标签的文本内容将自动更新。

当前仅支持解析为属性值或页面变量的简单表达式。将来可能会添加对涉及布尔或其他运算符的更复杂表达式的支持。

### 元素的静态属性

元素的静态属性的处理方式与静态属性元素类似，并使用类似的语法。例如，前面显示的用于演示静态属性元素的早期GridPane标记可以重写如下：
```xml
<GridPane>
    <children>
        <Label text="My Label" GridPane.rowIndex="0" GridPane.columnIndex="0"/>
    </children>
</TabPane>
```

除了更简洁之外，元素的静态属性（如实例属性）还支持位置、资源和变量解析运算符，唯一的限制是无法创建绑定到静态属性的表达式。

### 事件处理器属性

事件处理器属性是一种将行为附加到文档元素的便捷方式。任何定义setOnEvent ()方法的类都可以在标记中分配一个事件处理器，任何可观察的属性也可以（通过“onPropertyChange”属性）。

FXML 支持两种类型的事件处理器属性：脚本事件处理器和控制器方法事件处理器。下面分别讨论。

#### 脚本事件处理器

脚本事件处理器是在事件触发时执行脚本代码的事件处理器，类似于 HTML 中的事件处理器。例如，当用户按下按钮时，按钮的“onAction”事件的被执行，处理器使用JavaScript输出“You clicked me!” 文本到控制台：
```xml
<?language javascript?>
...

<VBox>
    <children>
        <Button text="Click Me!"
            onAction="java.lang.System.out.println('You clicked me!');"/>
    </children>
</VBox>
```

请注意在代码片段开头使用语言处理指令。此 指令 告诉 FXML 加载器应使用哪种脚本语言来执行事件处理器。每当在 FXML 文档中使用内联脚本时，都必须指定页面语言，并且每个文档只能指定一次。但是，这不适用于外部脚本，外部脚本可以使用任意数量的受支持脚本语言来实现。下一节将更详细地讨论脚本。

#### 控制器方法事件处理器

控制器方法事件处理器是由文档的“控制器”也就是Controller定义的方法。控制器是与 FXML 文档的反序列化内容相关联的对象，负责协调文档定义的对象（通常是UI元素）的行为。

控制器方法事件处理器由 “#” 符号指定，后跟处理器方法的名称。例如：
```xml
<VBox fx:controller="com.foo.MyController"
    xmlns:fx="http://javafx.com/fxml">
    <children>
        <Button text="Click Me!" onAction="#handleButtonAction"/>
    </children>
</VBox>
```

请注意在根元素上使用fx:controller属性。此属性用于将控制器类与文档相关联。如果MyController定义如下：
```java
package com.foo;

public class MyController {
    public void handleButtonAction(ActionEvent event) {
        System.out.println("You clicked me!");
    }
}
```

handleButtonAction ()将在用户按下按钮时被调用，文本“You clicked me!” 将被写入控制台。

通常，处理器方法应符合标准事件处理器的签名；也就是说，它应该采用扩展javafx.event.Event的类型的单个参数并且应该返回 void（类似于 C# 中的事件委托）。事件参数通常包含有关事件性质的重要且有用的信息；但是，它是可选的，如果不需要可以省略。

稍后部分将更详细地讨论控制器。

## 脚本

<fx:script> 标记允许调用者将脚本代码导入 FXML 文件或将脚本嵌入到 FXML 文件中。可以使用任何 JVM 脚本语言，包括 JavaScript、Groovy 和 Clojure 等。脚本代码通常用于直接在标记或关联的源文件中定义事件处理器，因为事件处理器通常可以用更松散类型的脚本语言编写，而不是用静态类型语言（如 Java）编写。

例如，以下标记定义了一个名为handleButtonAction()的函数，该函数由附加到Button元素的操作处理器调用：
```xml
<?language javascript?>

<?import javafx.scene.control.*?>
<?import javafx.scene.layout.*?>

<VBox xmlns:fx="http://javafx.com/fxml">
    <fx:script>
	    importClass(java.lang.System);

	    function handleButtonAction(event) {
	       System.out.println('You clicked me!');
	    }
    </fx:script>

    <children>
        <Button text="Click Me!" onAction="handleButtonAction(event);"/>
    </children>
</VBox>
```

单击该按钮会触发事件处理器，该事件处理器会调用该函数，产生与前面示例相同的输出。

脚本代码也可以在外部文件中定义。前面的示例可以拆分为一个 FXML 文件和一个 JavaScript 源文件，在功能上没有区别：

example.fxml
```xml
<?import javafx.scene.control.*?>
<?import javafx.scene.layout.*?>

<VBox xmlns:fx="http://javafx.com/fxml">
    <fx:script source="example.js"/>

    <children>
        <Button text="Click Me!" onAction="handleButtonAction(event);"/>
    </children>
</VBox>
```

example.js
```js
importClass(java.lang.System);

function handleButtonAction(event) {
   System.out.println('You clicked me!');
}
```

以这种方式将代码与标记分开通常更可取，因为许多文本编辑器支持为 JVM 支持的各种脚本语言突出显示语法。它还可以帮助提高源代码和标记的可读性。

请注意，脚本块不限于定义事件处理函数。脚本代码在处理时执行，因此它也可用于动态配置结果输出的结构。作为一个简单的示例，以下 FXML 包含一个脚本块，该脚本块定义了一个名为“labelText”的变量。此变量的值用于填充Label实例的文本属性：
```xml
<fx:script>
	var myText = "This is the text of my label.";
</fx:script>
...
<Label text="$myText"/>
```

## 控制器

虽然在脚本中编写简单的事件处理器可能很方便，无论是内联还是在外部文件中定义，但开发者通常更喜欢在强类型语言（如 Java）中书写更复杂的程序逻辑。如前所述，fx:controller属性允许调用者将“控制器”类与 FXML 文档相关联。控制器是一个编译类，它实现了文档定义的对象层次结构的“后台代码”。

如前所述，控制器通常用于为标记中定义的用户界面元素实现事件处理器：
```xml
<VBox fx:controller="com.foo.MyController"
    xmlns:fx="http://javafx.com/fxml">
    <children>
        <Button text="Click Me!" onAction="#handleButtonAction"/>
    </children>
</VBox>
```
```java
package com.foo;

public class MyController {
    public void handleButtonAction(ActionEvent event) {
        System.out.println("You clicked me!");
    }
}
```

在许多情况下，以这种方式简单地声明事件处理器就足够了。但是，当需要对控制器及其管理的元素的行为进行更多控制时，控制器可以定义一个initialize()方法，当其关联文档的内容已完全加载时，将在实施控制器上调用一次该方法：
```java
	public void initialize();
```

这允许实现类对内容执行任何必要的后处理。它还为控制器提供了访问fxml中用到的资源的途径，包括相对于fxml文档路径的位置（通常等同于文档本身的位置）。

例如，下面的代码定义了一个initialize()方法，它在代码中将一个动作处理器附加到一个按钮，而不是通过事件处理器属性，就像在前面的示例中所做的那样。按钮实例变量在读取文档时由加载程序注入。产生的应用程序行为是相同的：
```xml
<VBox fx:controller="com.foo.MyController"
    xmlns:fx="http://javafx.com/fxml">
    <children>
        <Button fx:id="button" text="Click Me!"/>
    </children>
</VBox>
```
```java
package com.foo;

public class MyController implements Initializable {
    public Button button;

    @Override
    public void initialize(URL location, Resources resources)
        button.setOnAction(new EventHandler<ActionEvent>() {
            @Override
            public void handle(ActionEvent event) {
                System.out.println("You clicked me!");
            }
        });
    }
}
```

### @FXML注解

请注意，在前面的示例中，控制器成员字段和事件处理器方法被声明为公共的，因此它们可以由加载程序设置或调用。一般情况下这不是问题，因为控制器通常只对创建它的 FXML 加载器可见。但是，对于不喜欢把控制器字段或处理器方法的可见性设置为public的开发人员，可以使用javafx.fxml.FXML注释。该注解将private或protect成员标记为可供 FXML 访问的。

例如，前面示例中的控制器可以重写如下：
```java
package com.foo;

public class MyController {
    @FXML
    private void handleButtonAction(ActionEvent event) {
        System.out.println("You clicked me!");
    }
}
```

```java
package com.foo;

public class MyController implements Initializable {
    @FXML private Button button;

    @FXML
    protected void initialize()
        button.setOnAction(new EventHandler<ActionEvent>() {
            @Override
            public void handle(ActionEvent event) {
                System.out.println("You clicked me!");
            }
        });
    }
}
```

在第一个版本中，handleButtonAction()带有@FXML标记，以允许fxml文档调用它。在第二个示例中，按钮字段添加了@FXML注解来允许fxmlLoader它的值。initialize ()方法也有类似的注解。

请注意，@FXML注释目前只能与受信任的代码一起使用。因为 FXML 加载器依赖于反射来设置成员字段和调用成员方法，所以它必须在任何非公共Field上调用setAccessible()。setAccessible()是一种特权操作，只能在安全上下文中执行。这可能会在未来的版本中改变。

### 嵌套控制器

通过<fx:include>元素加载的嵌套 FXML 文档的控制器实例直接映射到包含控制器的成员字段。这允许开发人员轻松访问由include定义的功能（例如应用程序的主窗口控制器显示的对话窗口）。例如，给定以下代码：

main_window_content.fxml
```xml
<VBox fx:controller="com.foo.MainController">
   <fx:include fx:id="dialog" source="dialog.fxml"/>
   ...
</VBox>
```

MainController.java
```java
public class MainController extends Controller {
    @FXML private Window dialog;
    @FXML private DialogController dialogController;
    ...
}
```

当调用控制器的initialize()方法时，dialog字段将被替换和赋值为从“dialog.fxml”加载的根元素，而dialogController字段将被赋值为dialog.fxml中定义的控制器对象。例如，主控制器可以调用include的子控制器上的方法来填充和显示对话框。

## FXML加载器

FXMLLoader类负责实际加载 FXML 源文件并返回生成的对象图。例如，以下代码从classPath路径上相对于加载类的位置加载 FXML 文件，并使用名为“com.foo.example”的资源包对其进行本地化。根元素的类型假定为javafx.scene.layout.Pane的子类，并且文档假定定义了一个类型为MyController的控制器：
```java
URL location = getClass().getResource("example.fxml");
ResourceBundle resources = ResourceBundle.getBundle("com.foo.example");
FXMLLoader fxmlLoader = new FXMLLoader(location, resources);

Pane root = (Pane)fxmlLoader.load();
MyController controller = (MyController)fxmlLoader.getController();
```

请注意， FXMLLoader#load()操作的输出是一个对象层次结构，它反映了文档中实际命名的类，而不是表示这些类的org.w3c.dom节点。在内部，FXMLLoader使用javax.xml.stream API（也称为_Streaming API for XML_或_StAX_）加载 FXML 文档。StAX 是一种极其高效的基于事件的 XML 解析 API，在概念上类似于其 W3C 前身 SAX。它允许单次处理 FXML 文档，而不是全量加载 DOM 结构到内存然后进行后处理。

### 自定义组件

FXMLLoader的setRoot ()和setController()方法允许调用者分别将文档根和控制器值注入文档命名空间，而不是将这些值的创建委托给FXMLLoader本身。这允许开发人员轻松创建可重复使用的控件，这些控件使用标记在内部实现，但（从 API 的角度来看）看起来与以编程方式实现的控件相同。

例如，以下标记定义了一个简单的自定义控件的结构，其中包含一个TextField和一个Button实例。根容器定义为javafx.scene.layout.VBox的实例：
```xml
<?import javafx.scene.*?>
<?import javafx.scene.control.*?>
<?import javafx.scene.layout.*?>

<fx:root type="javafx.scene.layout.VBox" xmlns:fx="http://javafx.com/fxml">
    <TextField fx:id="textField"/>
    <Button text="Click Me" onAction="#doSomething"/>
</fx:root>
```
如前所述，<fx:root>标记创建对先前定义的根元素的引用。该元素的值是通过调用FXMLLoader的getRoot()方法获得的。在调用load()之前，调用者必须通过调用setRoot()来指定此值。调用者可以类似地通过调用setController()为文档的控制器提供一个值，它设置读取文档时将用作文档控制器的值。在创建自定义的基于 FXML 的组件时，这两种方法通常一起使用。

在以下示例中，CustomControl类扩展了VBox （由<fx:root>元素声明的类型），并在其构造函数中将自身设置为 FXML 文档的根和控制器。加载文档时，CustomControl的内容将填充上一个 FXML 文档的内容：
```java
package fxml;

import java.io.IOException;

import javafx.beans.property.StringProperty;
import javafx.fxml.FXML;
import javafx.fxml.FXMLLoader;
import javafx.scene.control.TextField;
import javafx.scene.layout.VBox;

public class CustomControl extends VBox {
    @FXML private TextField textField;

    public CustomControl() {
        FXMLLoader fxmlLoader = new FXMLLoader(getClass().getResource("custom_control.fxml"));
        fxmlLoader.setRoot(this);
        fxmlLoader.setController(this);

        try {
            fxmlLoader.load();
        } catch (IOException exception) {
            throw new RuntimeException(exception);
        }
    }

    public String getText() {
        return textProperty().get();
    }

    public void setText(String value) {
        textProperty().set(value);
    }

    public StringProperty textProperty() {
        return textField.textProperty();
    }

    @FXML
    protected void doSomething() {
        System.out.println("The button was clicked!");
    }
}
```
现在，调用者可以在代码或标记中使用此控件的实例，就像任何其他控件一样；例如：

java
```java
HBox hbox = new HBox();
CustomControl customControl = new CustomControl();
customControl.setText("Hello World!");
hbox.getChildren().add(customControl);
```

fxml
```xml
<HBox>
    <CustomControl text="Hello World!"/>
</HBox>
```

[copyright](http://docs.oracle.com/javase/7/docs/legal/cpyr.html)(c) 2008, 2014, Oracle and/or its affiliates. All rights reserved.

翻译来源：[https://docs.oracle.com/javafx/2/api/javafx/fxml/doc-files/introduction_to_fxml.html](https://docs.oracle.com/javafx/2/api/javafx/fxml/doc-files/introduction_to_fxml.html)