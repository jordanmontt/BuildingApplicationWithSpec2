## Layouts
@cha_layout

status: to be finished
status: spellchecker

In Spec2 layouts are represented by instances of layout classes. Such layout classes encode different positioning of elements such as box, paned, or grid layouts.
This chapter presents the existing layouts, their definition, and how layouts can be reused when 
a presenter reuses other presenters.

### Basic principle reminder

Spec expects to get layouts objects, instances of the layout classes, associated with a presenter. Each presenter should describe the positioning of its sub-presenters. 

Contrary to Spec1.0 where layouts were only defined at the class level, in Spec2.0
to define the layout of a presenter you can: 
- Define the `defaultLayout` method on the instance side,
- Or use the message `layout:` in your `initializePresenters` method to set an instance of layout in the current presenter.

Both layout methods should return a layout, for example, instance of `SpBoxLayout` or `SpPanedLayout`. These two methods are the preferred way to define layouts.

Note the possibility of defining a class-side accessor e.g. `defaultLayout` will remain for those who prefer it.

This new design reflects the dynamic nature of layouts in Spec2, and the fact that you can compose them using directly presenter instances, not forcing you to declare upfront sub-presenters in instance variables and then use their names as it was done in Spec1.0.
It is, however, possible that there are cases where you want a layout "template"... so you still can do it.


### A running example

To be able to play with the layouts defined in this chapter, 
we define a simple presenter named `SpTwoButtons`.
 
``` 
SpPresenter << #SpTwoButtons 
    slots: { #button1 . #button2 }; 
    package: 'CodeOfSpec20BookThreePillar'
```

We define a simple `initializePresenters` method as follows: 

```
SpTwoButtons >> initializePresenters 
    button1 := self newButton. 
    button2 := self newButton. 

    button1 label: '1'.
    button2 label: '2'.
```



### BoxLayout (SpBoxLayout and SpBoxConstraints)

The class `SpBoxLayout` displays presenters in an ordered sequence of boxes. 
A box can be horizontal or vertical and presenters are ordered top to down or left to right following the direction decided. A box layout can be composed of other layouts.


![Two buttons placed horizontally from left to right.](figures/TwoButtonsLeftToRight.png width=50&label=TwoButtonsLeftToRight) 

Let us define a first simple layout whose result is displayed in  Fig. *@TwoButtonsLeftToRight@* as follows.

```
SpTwoButtons >> defaultLayout
     ^ SpBoxLayout newLeftToRight
        add: button1; 
        add: button2; 
        yourself
```

What we see is that by default a presenter expands its size to fit the space of its container. 

An element in a vertical box will use all available horizontal space, and fill
vertical space according to the rules. This is inversed in a horizontal box.

We can refine this layout to indicate that the sub-presenters should not expand to their container using the message `add:expand:`. The result is shown in Figure *@TwoButtonsLeftToRightExpanded@*.

```
SpTwoButtons >> defaultLayout
    ^ SpBoxLayout newLeftToRight
        add: #button1 expand: false ; 
        add: #button2 expand: false ;
        yourself
```

![Two buttons placed horizontally from left to right but not expanded.](figures/TwoButtonsLeftToRightNotExpanded.png width=50&label=TwoButtonsLeftToRightExpanded) 

The full message to add presenters is: `add:expand:fill:padding:`
- `expand:` argument - when true, the new child is to be given extra space allocated to the box. The extra space is divided evenly between all children that use this option.
- `fill:` argument - when true, the space given to a child by the expand option is actually allocated to the child, rather than just padding it. This parameter has no effect if `expand` is set to `false`.
- `padding:` argument  - extra space in pixels to put between this child and its neighbors, over and above the global amount specified by “spacing” property. If a child is a widget at one of the reference ends of the box, then padding pixels are also put between the child and the reference edge of the box.


To illustrate a bit this API, we add another button to the presenter and change the `defaultLayout` method as follows. 
The result is shown in Fig *@ThreeButtons@*. 
We want to stress however that it is better not to use a fixed width or padding.

```smalltalk
SpTwoButtons >> defaultLayout 
    ^ SpBoxLayout newTopToBottom
        spacing: 15;
        add: button1 expand: false fill: true padding: 5;
        add: button2 withConstraints: [ :constraints | constraints width: 30; padding: 5];
        addLast: button3 expand: false fill: true padding: 5;
    yourself
```


![Three buttons placed from top to bottom playing with padding and fill options.](figures/ThreeButtons.png width=50&label=ThreeButtons) 



### Example setup for layout reuse

Before presenting some of the other layouts, we show an important aspect of Spec presenter composition: a composite can declare that it wants to reuse a presenter using a specific layout of such a presenter. 

Consider our artificial example of a two-button UI. Let us define two layouts as follows:
We define two class methods returning different layouts. Note that we could define such methods on the instance side too and we define them on the class side to be able to get such layouts without an instance of the class.

``` 
SpTwoButtons class >> buttonRow 
    ^ SpBoxLayout newLeftToRight 
        add: #button1; add: #button2; 
        yourself 
``` 
 
``` 
SpTwoButtons class >> buttonCol 
    ^ SpBoxLayout newTopToBottom 
        add: #button1; add: #button2; 
        yourself 
``` 
 
Note that when we define the layout at the class level, we use a symbol whose name is the corresponding instance variable. Hence we use `#button2` to refer to the presenter stored in the instance variable `button2`. This mapping can be customized at the level of the presenter but we do not present this because we never got the need for it. 


### Opening with a layout

Spec message `openWithLayout:` lets you specify the layout you want to use to open the presenter.
Here are some examples:
- `SpTwoButtons new openWithLayout: SpTwoButtons buttonRow` places the buttons in a row. 
- `SpTwoButtons new openWithLayout:  SpTwoButtons buttonCol` places them in a column. 



We define a `defaultLayout` method just invoking one of the previously defined methods.
``` 
SpTwoButtons >> defaultLayout 
    ^ self class buttonRow 
```

We define also a `defaultLayout` method so that the presenter can be opened without defining a given layout. 


### Better design

Now we can do better and define two instance level methods to encapsulate the layout configuration. 

```
SpTwoButtons >> beCol
    self layout: self class buttonCol
```

```
SpTwoButtons >> beRow
    self layout: self class buttonRow
```

We can then write the following script:

```
SpTwoButtons new 
    beCol;
    open 
```


### Specifying a layout when reusing a presenter 

Having multiple layouts for a presenter implies that there is a way to specify the layout to use when a presenter is reused. 
This is simple we use the method `layout:`.
Here is an example. 
We create a new presenter named: `SpButtonAndListH`.
 
``` 
SpPresenter << #SpButtonAndListH 
    slots: { #buttons . #list }; 
    package: 'CodeOfSpec20BookThreePillar'
```


```
SpButtonAndListH >> initializePresenters 
    buttons := self instantiate: SpTwoButtons. 
    list := self newList. 
    list items: (1 to: 10). 
``` 

``` 
SpButtonAndListH >> initializeWindow: aWindowPresenter 
    aWindowPresenter title: 'SuperWidget' 
```

```
SpButtonAndListH >> defaultLayout 
    ^ SpBoxLayout newLeftToRight 
          add: buttons;
          add: list; 
          yourself 
```


This `SpButtonAndListH ` class results in a SuperWidget window as shown in Figure *@fig_alternativeButton@*.  
It reuses the `SpTwoButtons` widget and places all three widgets in a horizontal order because the `SpTwoButtons` widget will use the `buttonRow` layout method. 
 
![Screen shot of the UI with buttons placed horizontally](figures/alternativeButton.png width=50&label=fig_alternativeButton) 
 
Alternatively, we can create `TBAndListV` class as a subclass of `SpButtonAndListH ` and only change the `defaultLayout` method as below.
It specifies that the reused `buttons` widget should use the `buttonCol` layout method, and hence results in the window shown in Figure *@fig_SuperWidget@*.


![Screen shot of the UI with buttons placed vertically](figures/SuperWidget.png width=50&label=fig_SuperWidget) 

``` 
SpButtonAndListH << #TButtonAndListV 
    package: 'CodeOfSpec20BookThreePillar'
``` 
 
```
TButtonAndListV >> initializePresenters
    super initializePresenters.
    buttons beCol
```

##### Alternative to declare subcomponent layout choice.

The alternative is to define a new method `defaultLayout` and to use the `add:layout:`. 
We define a different presenter.

```
SpButtonAndListH << #TButtonAndListV2
    package: 'CodeOfSpec20BookThreePillar'
```

We define a new `defaultLayout` method as follows: 

``` 
SpButtonAndListV2 >> defaultLayout
    ^ SpBoxLayout new
        add: buttons layout: #buttonCol;
        add: list;
        yourself
```

Note the use of the method `add:layout:` with the selector of the method returning the layout configuration 
here #buttonCol. This is normal since we cannot access the state of a subcomponent at this moment.

##### Dynamically changing a layout
It is possible to change the layout of a presenter dynamically for example from the inspector as shown in *@figTweak@*.

![Tweaking and playing interactively with layouts from the inspector.](figures/Interactive.png width=100&label=figTweak) 



### GridLayout 

The class `SpGridLayout` arranges sub-presenters in a grid according to its properties (position and
span), and according to certain layout properties such as:
- A position is mandatory (`columnNumber@rowNumber`)
- A span can be added if desired (`columnExtension@rowExtension`)

The following example 

```
SpPresenter << #SpGridExample
	slots: {#nameText . #passwordText . #acceptButton . #cancelButton};
	package: 'CodeOfSpec20BookThreePillar'
```

```
SpGridExample >> initializePresenters
    nameText := self newTextInput.   
    passwordText := self newTextInput.
    acceptButton := self newButton.
    acceptButton label: 'Accept'.
    cancelButton := self newButton.
    cancelButton label: 'Cancel'.
```


```smalltalk
SpGridExample >> defaultLayout
    ^ SpGridLayout new
        add: 'Name:' at: 1@1;
        add: #nameText  at: 2@1;
        add: 'Password:' at: 1@2;
        add: #passwordText at: 2@2;  
        add: #acceptButton at: 1@3;
        add: #cancelButton at: 2@3 span: 2@3;
        add: 'test label' at: 1@4;
        yourself
```

![An ugly example .](figures/grid.png width=90&label=grid) 

Here is a list of options: 
- `columnHomogeneous`: Whether a column will have the same size.
- `rowHomogeneous`: Whether a row will have the same size.
- `colSpacing:`: The column space between cells.
- `rowSpacing:`: The row space between cells.



### Paned layout (SpPanedLayout and SpPanedConstraints)

A paned layout is like a Box Layout (it places children in a vertical or horizontal
fashion), but it will add a splitter in between, that the user can drag to resize the panel.
In exchange, a paned layout can have just two children. Position indicates
the original position of the splitter. It can be nil (then it defaults to 50%) or It can
be a percentage (e.g. 30 percent)

```smalltalk
SpPanedLayout newHorizontal position: 80 percent;
    add: acceptButton;
    add: cancelButton;
    yourself.
```

        
        
### Overlay


```
app := SpApplication new.
app addStyleSheetFromString: '.application [
        .green [ 
            Draw {
                #backgroundColor: #16A085
            }
        ],
        .redOverlay [
            Draw { #backgroundColor: #C0392BBB },
            Geometry { #height: 150, #width: 150 }
        ],
        .title [ Font { #size: 40, #bold: true },
            Geometry { #height: Reset, #width: Reset } ]
]'.

presenter := SpPresenter newApplication: app.
    
child := presenter newPresenter
     layout: (SpBoxLayout new
         hAlignCenter;
         vAlignCenter;
        add: ('I AM THE CHILD' asPresenter
                  addStyle: 'title';
                yourself);
            yourself);
          addStyle: 'green';
        yourself.
        
overlay := presenter newPresenter
    layout: SpBoxLayout newVertical;
    addStyle: 'redOverlay';
    yourself.

presenter layout: (SpOverlayLayout new
     child: child;
     addOverlay: overlay withConstraints: [ :c | 
         c
             vAlignCenter;
             hAlignCenter ];
     yourself).
            
presenter open.

```

### Conclusion

Spec offers several predefined layouts. New ones will probably be added but in compatible way.
An important closing point is that layouts can be dynamically composed. It means that you will  be able to 
design applications that can adapt to specific conditions.


