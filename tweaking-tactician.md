---
layout: default
permalink: /tweaking-tactician/
title: Tweaking Tactician
---

# Tweaking Tactician
Tactician is really flexible, you can change almost anything about it. This page walks you through setting up a standard Command Bus that executes Commands by passing them to a matching Handler.

Along the way, we'll show you the important parts of Tactician so you can tweak them yourself.

## 1. Handler Method
First, what method do you want to call on your Handlers when they receive a command? This is determined by the [`MethodNameInflector` classes](https://github.com/thephpleague/tactician/tree/master/src/Handler/MethodNameInflector). 

Assuming we have a Command class called `RentMovieCommand`, this table shows you what each inflector does: 

Handler Method             | Class to use                   | Notes
---------------------------|--------------------------------|--------------------------------------------------------
`handleRentMovieCommand()` | **`HandleClassNameInflector`** | Lets you handle multiple commands on the same class
`handle()`                 | **`HandleInflector`**          | Easy to read, easy to guess
`__invoke()`               | **`InvokeInflector`**          | Good for invokable classes or closures.

If you'd like to do something custom here, just implement the [`MethodNameInflector` interface](https://github.com/thephpleague/tactician/blob/master/src/Handler/MethodNameInflector/MethodNameInflector.php). All you need to do is return the string name of the method to call. This inflector will also receive the command and handler so you can use Reflection on them if you want.

## 2. Loading Your Handlers
Next, you need to choose how your Handlers are loaded and mapped to incoming commands. This is done by the [`HandlerLocator` classes](https://github.com/thephpleague/tactician/tree/master/src/Handler/Locator). 

If you're using a Tactician framework module/bundle/provider, you should probably use the custom loader for that framework, but any locator will work.

The complete list of options are:

Locator           | Notes
------------------|---------------------------------------------------
[`InMemoryLocator`](https://github.com/thephpleague/tactician/blob/master/src/Handler/Locator/InMemoryLocator.php) | Matches Command class name to a Handler instance.

Configuring the Locator will vary depending on which Locator you use. Some will work with DI containers, some just use plain PHP arrays, so check the class itself.

Custom Locators only need to implement the [HandlerLocator interface](https://github.com/thephpleague/tactician/blob/master/src/Handler/Locator/HandlerLocator.php). This is just a single method that receives the Command and returns the Handler.

## 3. Creating the Command Bus
Now that you've chosen a Locator and MethodNameInflector, you need to pass them to the Command Bus for execution. In Tactician, there's one "core" command bus that always loads and executes the Handlers: the [HandlerCommandBus](https://github.com/thephpleague/tactician/blob/master/src/HandlerCommandBus.php).

Pass the Locator and MethodNameInflector to that, and you've got a Command Bus to use in your app.

~~~ php
// Choose our method name
$inflector = new HandleClassNameInflector();

// Choose our locator and register our command
$locator = new InMemoryLocator();
$locator->addHandler(new RentMovieHandler(), RentMovieCommand::class);

// Create the command bus
$commandBus = new HandlerCommandBus($locator, $inflector);
~~~

## 4. Decorators
Now that you've got the core Command Bus working, you can customize it further by adding decorators that change the behavior when it's called.

For example, we recommend using the LockingCommandBus, since that will prevent one command from being executed while another is already running.

Building on our previous example:

~~~ php
// We wrap our command bus in a new one that adds this locking behavior.
$commandBus = new LockingCommandBus($commandBus); 
~~~

You can create your own custom decorators by implementing the [CommandBus interface](https://github.com/thephpleague/tactician/blob/master/src/CommandBus.php).

Tactician aims to ship with lots of useful decorators, you can find a complete list in the menu.

### And more
If you've read this far down but you're still curious, take a look at the [examples directory on Github](https://github.com/thephpleague/tactician/tree/master/examples).

Also, if you've implemented something custom on this page, from Inflectors to Decorators, please send us a pull request so we can share it with other Tactician users. We'd really appreciate it.