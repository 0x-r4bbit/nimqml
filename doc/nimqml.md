:Authors:
	Filippo Cucchetto <filippocucchetto@gmail.com>

	Will Szumski <will@cowboycoders.org>
:Version: 0.4.5
:Date: 2015/09/15

Introduction
-----------
The NimQml module adds Qt Qml bindings to the Nim programming language
allowing you to create new modern UI by mixing the Qml declarative syntax
and the Nim imperative language.

The NimQml is made by two components:
* The DOtherSide C++ shared library
* The NimQml Nim module

This first component implements the glue code necessary for
communicating with the Qt C++ library, the latter module wraps
the libDOtherSide exported symbols in Nim

Building
--------
At the time of writing the DOtherSide C++ library must be compiled
and installed manually from source.

First clone the DOtherSide git repo
::
  git clone https://github.com/filcuc/DOtherSide

than you can proceed with the common CMake build steps

::
  mkdir build
  cd build
  cmake ..
  make

If everything goes correctly, you'll have built both
the DOtherSide C++ library and the Nim examples

Installation
----------
The installation is not mandatory, in fact you could try
the built Nim example in the following way
::
  cd path/to/build/dir
  cd Nim/Examples/HelloWorld
  export LD_LIBRARY_PATH=path/to/libDOtherSide.so
  ./HelloWorld

The DOtherSide project is made of two components
1. The DOtherSide C++ lib
2. The NimQml module

You can procede with the installation of the C++ library
in the following way
::
  cd to/build/dir
  make install
or by manually copying the library in your system lib directory
::
  sudo cp build/dir/path/DOtherSide/libDOtherSide.so /usr/lib

For the NimQml module you can use the ``nimble`` package manager
::
  nimble install NimQml

or
::
  cd to/build/dir/Nim/NimQml
  nimble install


Example 1: HelloWorld
----------
As usual lets start with an HelloWorld example.
Most of the NimQml projects are made by one or more nim and qml
files. Usually the .nim files contains your app logic and data
layer. The qml files contain the presentation layer and expose
the data in your nim files.

``Examples/HelloWorld/main.nim``

.. code-block:: nim
   :file: ../Examples/HelloWorld/main.nim

``Examples/HelloWorld/main.qml``

.. code-block:: qml
   :file: ../Examples/HelloWorld/main.qml

The following example shows the basic steps of each NimQml app
1. Create the QApplication for initializing the Qt runtime
2. Create the QQmlApplicationEngine and load your main .qml file
3. Call the ``exec`` proc of the QApplication instance for starting
the Qt event loop

Example 2: exposing data to Qml
------------------------------------
The previous example shown you how to create a simple application
window and how to startup the Qt event loop.

It's time to explore how to pass data to Qml, but lets see the
example code first:

``Examples/SimpleData/main.nim``

.. code-block:: nim
   :file: ../Examples/SimpleData/main.nim

``Examples/SimpleData/main.qml``

.. code-block:: qml
   :file: ../Examples/SimpleData/main.qml

The following example shows how to expose simple data types to Qml:
1. Create a QVariant and set its internal value.
2. Create a property in the Qml root context with a given name.

Once a property is set through the ``setContextProperty`` proc, it's available
globally in all the Qml script loaded by the current engine (see the official Qt
documentation for more details about the engine and context objects)

At the time of writing the QVariant class support the following types:
* int
* string
* bool
* float
* QObject derived classes

Example 3: exposing complex data and procedures to Qml
----------------------------------------------------------
As seen by the second example, simple data is fine. However most
applications need to expose complex data, functions and
update the view when something changes in the data layer.
This is achieved by creating an object that derives from QObject.

A QObject is made of :
1. ``Slots``: slots are functions that could be called from the qml engine and/or connected to Qt signals
2. ``Signals``: signals allow the sending of events and be connected to slots
3. ``Properties``: properties allow the passing of data to
   the Qml view and make it aware of changes in the data layer

A QObject property is made of three things:
* a read slot: a method that returns the current value of the property
* a write slot: a method that sets the value of the property
* a notify signal: emitted when the current value of the property is changed

We'll start by looking at the main.nim file

``Examples/SlotsAndProperties/main.nim``

.. code-block:: nim
   :file: ../Examples/SlotsAndProperties/main.nim

Here, nothing special happens except:
1. The creation of Contact object
2. The injection of the Contact object to the Qml root context
   using the ``setContextProperty`` as seen in the previous
   example

The Qml file is as follows:

``Examples/SlotsAndProperties/main.qml``

.. code-block:: qml
   :file: ../Examples/SlotsAndProperties/main.qml

The qml is made up of: a Label, a TextInput widget, and a button.
The label displays the contact name - this automatically updates when
the contact name changes.

When clicked, the button updates the contact name with the text from
the TextInput widget.

So where's the magic?

The magic is in the Contact.nim file

``Examples/SlotsAndProperties/Contact.nim``

.. code-block:: nim
   :file: ../Examples/SlotsAndProperties/Contact.nim

First we declare a QObject subclass and provide a simple
new method where we:
1. invoke the ``create()`` procedure. This invoke the C++ bridge and allocate
a QObject instance
2. register a slot ``getName`` for reading the Contact name field
3. register a slot ``setName`` for writing the Contact name
4. register a signal ``nameChanged`` for notify the contact name changes
5. register a property called ``name`` of type ``QString`` with the given
   read, write slots and notify signal

Looking at the ``getName`` and ``setName`` methods, you can see that slots, as defined in Nim,
are nothing more than standard methods. The method corresponding to the ``setName`` slot
demonstrates how to use the ``emit`` method to emit a signal.

The last thing to consider is the override of the ``onSlotCalled`` method.
This method is called by the NimQml library when an invocation occurs from
the Qml side for one of the slots belonging to the QObject.
The usual implementation for the onSlotCalled method consists of a
switch statement that forwards the arguments to the correct slot.
If the invoked slot has a return value, this is always in the index position
0 of the args array.


Example 4: QtObject macro
-------------------------
The previous example shows how to create a simple QObject, however writing
all those ``register`` procs and writing the ``onSlotCalled`` method
becomes boring pretty soon.

Furthermore all this information can be automatically generated.
For this purpose you can import the NimQmlMacros module that provides
the QtObject macro.

Let's begin as usual with both the main.nim and main.qml files

``Examples/QtObjectMacro/main.nim``

.. code-block:: nim
   :file: ../Examples/QtObjectMacro/main.nim


``Examples/QtObjectMacro/main.qml``

.. code-block:: qml
   :file: ../Examples/QtObjectMacro/main.qml

Nothing is new in both the ``main.nim`` and ``main.qml`` with respect to
the previous example. What changed is the Contact object:

``Examples/QtObjectMacro/Contact.nim``

.. code-block:: nim
   :file: ../Examples/QtObjectMacro/Contact.nim

In details:
1. Each QObject is defined inside the QtObject macro
2. Each slot is annotated with the ``{.slot.}`` pragma
3. Each signal is annotated with the ``{.signal.}`` pragma
4. Each property is created with the ``QtProperty`` macro

The ``QtProperty`` macro has the following syntax

.. code-block:: nim
  QtProperty[typeOfProperty] nameOfProperty

Example 5: ContactApp
-------------------------
The last example tries to show you all the stuff presented
in the previous chapters and gives you an introduction to how
to expose lists to qml.

Qt models are a huge topic and explaining how they work is
out of scope. For further information please read the official
Qt documentation.

The main file follows the basic logic of creating a qml
engine and exposing a QObject derived object "ApplicationLogic"
through a global "logic" property

``Examples/ContactApp/main.nim``

.. code-block:: nim
   :file: ../Examples/ContactApp/main.nim

The qml file shows a simple app with a central tableview

``Examples/ContactApp/main.qml``

.. code-block:: qml
   :file: ../Examples/ContactApp/main.qml

The important things to notice are:
1. The menubar load, save and exit items handlers call the logic load, save and exit slots
2. The TableView model is retrieved by the logic.contactList property
3. The delete and add buttons call the del and add slots of the logic.contactList model

The ApplicationLogic object is as follows:

``Examples/ContactApp/ApplicationLogic.nim``

.. code-block:: nim
   :file: ../Examples/ContactApp/ApplicationLogic.nim

The ApplicationLogic object,
1. expose some slots for handling the qml menubar triggered signals
2. expose a contactList property that return a QAbstractListModel derived object that manage the list of contacts

The ContactList object is as follows:

``Examples/ContactApp/ContactList.nim``

.. code-block:: nim
   :file: ../Examples/ContactApp/ContactList.nim

The ContactList object:
1. overrides the ``rowCount`` method for returning the number of rows stored in the model
2. overrides the ``data`` method for returning the value for the exported roles
3. overrides the ``roleNames`` method for returning the names of the roles of the model. This name are then available in the qml item delegates
4. defines two slots ``add`` and ``del`` that add or delete a Contact. During this operations the model execute the ``beginInsertRows`` and ``beginRemoveRows`` for notifing the view of an upcoming change. Once the add or delete operations are done the model execute the ``endInsertRows`` and ``endRemoveRows``.
