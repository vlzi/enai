# enai
This documentation explains how to write a valid Enai code.

Enai is a js-based environment to write codes. All you need to do is to call js constructors or member functions. Then it will generate Entry codes. Entry is a programming language supporting visual scenes, objects, signal, ets. Then Enai will compile your js code to Entry file.

Caution: while you are basically writing JS code, I don't recommend using JS control statement, modules, etc. They will not be converted into final code. Instead you can use flow statements explained below. Moreover, If you have no particular intention, use comment statement instead of JS comment.

# Import
First you should write
```JS
const { Project } = require("./enai");
```
and create a project by calling `new Project(project name)`.

Example:
```JS
const project = new Project("My Entry Project");
```

# Scene
Scene is a room for objects to move around on. You can create a scene by calling `project.Scene(title)`.

Example:
```JS
const project = Project("RPG");
const lobby = project.Scene("Lobby"), game_title = project.Scene("Title");
```
Scenes have size of 480x270. The center is (0, 0) and the upper-right corner is (240, 135). Note that switching scene makes every running code on current scene stop.

# Object
Object is a moving entity which has property x, y, rotation, direction, etc. Switching scenes will reset these values. You can create an object by calling `scene.Object(object options)`

Example:
```JS
const project = Project("game");
const game_title = project.Scene("Title");
const startButton = game_title.Object({ name: "Start button" });
```
An object should belong to exactly one scene.
property | type | explaination
--- | --- | ---
name | string | the name of object. irrelevant with final operation.
objectType | "sprite" or "textBox" | the type of object. Only if this value is "textBox", the object can run textBox statements.
rotation | number | the orientation of object in degrees. Increasing this value rotates object counterclockwise.
font | "{number}px" | font size of the object with type of textBox. If the type is "sprite", this property is not used.
scaleX | number | how much the object is stretched sideways. Real width shown to user is `(current picture width) * scaleX`.
scaleY | number | how much the object is stretched vertically. Real height shown to user is `(current picture height) * scaleY`.
visible | boolean | whether the object is shown or not.
x | number | x-coordinate of center of the object. The absolute value of this value can exceed 240. In this case, part or all of the object is invisible.
y | number | y-coordinate of center of the object. The absolute value of this value can exceed 135. In this case, part or all of the object is invisible.

# Picture / Sound
You can create a picture by calling `object.picture(file path)` and sound by calling `object.sound(file path)`
Example:
```JS 
const project = Project("My game");
const game_title = project.Scene("Title");
const startButton = game_title.Object({ name: "Start button" });
startButton.sound("./click_sound.mp3");
startButton.picture("./start_button.png");
```
Resources added to object can be used by statements only in that object.

# Appendable
Appendable refers to event, function, or body of flow statements. Appendable objects have member function, which allows you to append statement. The name of statements are listed [below](#event--statement--expression)


# Events / Statements / Expressions
Event lets you run statements when you want. Statement is an JS Object corresponding to a line of other languages. You can create an event by calling `object.event_name(event_parameters)`, append
a statement by calling `event.statement_name(statement_parameters)` and use expression in it by calling `project.expression_name(expression_parameters)`. If a statement takes other statements, write them between `.ƹ .ʒ`. An example is below.
Example:
```JS
const project = Project("HI");
const game_title = project.Scene("Title");
const startButton = game_title.Object({ name: "Start button" });
startButton.when_some_key_pressed(32)
	.dialog("You pressed space key.", "speak")
	.comment("This statement speaks that the user pressed space key.")
	.if(project.boolean_basic_operator(3, "EQUAL", 2)).ƹ
		.dialog(project.calc_basic(3, "PLUS", 2), "think")
	.ʒ.else.ƹ
		.if(project.boolean_basic_operator(3, "EQUAL", 4)).ƹ
			.start_neighbor_scene("next")
		.ʒ
	.ʒ
```
While you can create multiple events with the same types, it is not gueranteed that subsequent statements in an object are evaluated delaying/awaiting other events in the same object or in the same scene. Moreover, I don't know the evaluation order of statements with same type of events. Note that both `object.event_name` and `event.statement_name` returns event. Event and statement constructor types are listed below. Every type `type` below can be replaced with expression of that type. For example, `project.calc_basic(1, "PLUS" 2)` or `project.combine_something("hello", "!!!")` can be substituted into `string | number`.

## Events
* `when_some_key_pressed(keycode: number | string): event` creates an event runned when the key with the keycode is pressed.

* `when_run_button_click(): event` creates an event runned when the Entry engine is started.

## Statements:
### Comment
* `appendable.comment(c: string): appendable` append comment about the statement right before comment statement. If this is first statement, then it is about the event it belongs to.
### Start
* `appendable.start_neighbor_scene(direction: string): appendable` switches to next or previous scene. `direction` should be either `"next"` or `"prev"`, respectively next and previous scene. If you append other statement after this, it is not sure wheter that statement is evaluated or not.
### Flow
* `appendable.repeat_basic(count: integer, (repeat: Appendable) => any)): appendable` repeats statements appended to `repeat` `count` times.

* `appendable._if(condition: boolean, (then: Appendable)=>any): appendable` runs statements appended to `then` if `condition` is true.

* `appendable.if_else(condition: boolean, (then: Appendable)=>any, (else: Appendable) => any): appendable` runs statements appended to `then` if `condition` is true, statements appended to `else` if `condition` is false.

Note that every repetition statements has one frame delay per iterations. To repeat statements as fast as possible, consider using recursive function. Unfortunately, Entry does not support trigonal operator. It is recommended creating Entry global variable and using it instead. IMPORTANT: you can not use Appendable object as a repeat count! Every repeat statement does not support current iteration count, so you have to make global Entry variable and increase it manually in this case if you need.

### Shape
* `appendable.dialog(content: string | number, dial_type: string): appendable` makes speech bubble with the type of dial_type and content of `content`. `dial_type` should be either `"speak"` or `"think"`.

### Variable / List
* `appendable.ask_and_wait(content: string | number): appendable` asks user question saying `content`, and await its response. Or set canvas input value to first line inputed through console / standard input that is not processed yet. If multiple question asked in the same time and the same project, question asked later is queued. As this is a statement, this returns nothing. To retrieve answer, use `project.get_canvas_input_value()`.

* `appendable.set_variable(target: variable, value: string | number): appendable` sets `target` to `value`.

* `appendable.add_value_to_list(target: list, value: string | number): appendable` pushes `value` to `target`.

* `appendable.change_value_list_index(target: list, index: number, value: string | number)` sets the `index`-th item of `target` to `value`. `index` is one-based. Don't get confused with zero-based-indexing!

## Expressions
When this documentation says a expression returns `something`, it is not saying `project.expression_name(params)` returns `something`. It returns Entry expression returning `something`.

### Judgement
* `project.boolean_basic_operator(a: number, comp: string, b: number): expression(boolean)` compares `a` with `b`. If `comp` is `"EQUAL"`, returns whether `a` is equal to `b`. If `comp` is `"NOT_EQUAL"`, returns whether `a` is not equal to `b`. If `comp` is `"GREATER"`, returns whether `a` is greater than `b`. If `comp` is `"LESS"`, returns whether `a` is less than `b`. If `comp` is `"GREATER_OR_EQUAL"`, returns whether `a` is not less than `b`. If `comp` is `"LESS_OR_EQUAL"`, returns whether `a` is not greater than `b`.
### Calculation
* `project.calc_basic(a: number, operator: string, b: number): expression(number)` calculates arithmetic operations. If `operator` is `"PLUS"`, returns `a + b`. If `operator` is `"MINUS"`, returns `a - b`. If `operator` is `"MULTI"`, returns `a * b`. If `operator` is `"PLUS"`, returns `a + b`. If `operator` is `"DIVIDE"`, returns `a / b` in floating point number.

* `project.combine_something(a: string, b: string): expression(string)` concatenates `a` and `b` and returns it.

### Variable / List
* `project.get_canvas_input_value(): expression(string | number)` returns the most recent question response.

* `project.get_variable(v: variable): expression(number | string)` returns the value of `variable`.

* `project.value_of_index_from_list(l: list, index: number): expression(number | string)` returns the value of the `index`-th item of `l`. `index` is 1-based index.

# Variable / List
Unlike other languages, Entry distinguish variable and list. Variables can hold only one value, which is string or number. List can hold multiple values, samely string or number. To make a variable, call `project.Variable(name)`. To make a list, `project.List(name)`. As you can see, there is only global variable / list in Entry. Manipulate variable or list using `appendable.manipulatingStatement(variableOrList, [ additionsParams ])`. You can get expression representing value simply by `project.get_variable(variable)` or `project.value_of_index_from_list(list, one-based-index)`.

Example:
```JS
const numbers = project.List("numbers"), result = project.Variable("result");
const calculatorScene = project.Scene("Calculator");
const calculator = calculatorScene.Object("Calculator");
calculator.when_object_click()
	.add_value_to_list(numbers, 3)
	.add_value_to_list(numbers, 5)
	.change_value_list_index(numbers, 2, 4)
	.set_variable(result, calc_basic(
		project.value_of_index_from_list(numbers, 1),
		"PLUS",
		project.value_of_index_from_list(numbers, 2)
	))
	.comment("The value of result is now 7.")
	.dialog(project.get_variable(result), "speak")
```

# Finish your project!
you can save your project by calling `project.save(path)`.
Example:
```JS
const project = new Project("My Entry Project");
project.save("./my entry project.ent");
```

# Tips
* Loop count is not supported. You cannot use callback Appendable as loop counter. If you need, you should define global variable, and increase/decrease it manually.

Don't do this:
```JS
object.when_run_button_click()
	.comment("The code below will raise compile error.")
	.repeat_basic(10, loop=>loop
		.dialog(project.calc_basic(10, "MINUS", loop) "speak")
	)
```
But do this:
```JS
const count = project.Variable("Loop count");
object.when_run_button_click()
	.set_variable(count, 10)
	.repeat_basic(10, loop=>loop
		.dialog(project.get_variable(count), "speak")
		.set_variable(count, project.calc_basic(project.get_variable(count), "MINUS", 1))
	)
```
* Indexing is one-based. Don't get confused. I don't recommend adding or substracting one to convert indexing. Try minimizing it.
* Ternary operator is not supported. use if_else statement instead.
* If you think some syntaxes which should exist don't exist, Don't create new syntax using what you are knowing. Syntaxes explained here can only be operated how they are explained. Rather, ask me to add a new syntax. 
* Comment using appendable.comment statement. One line: `appendable.comment("one line")` Multi lines:
```JS
appendable.comment(`
	line 1
	line 2
`)
```
* Even when writing codes performing simple arbitary opration, think when it should be performed. If it should be performed on final Entry file, then you should use `project.calc_basic(a, "MINUS", b)` instead of `a - b`.
