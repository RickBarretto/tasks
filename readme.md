<h1 align="center">
    Tasks
</h1>

<p align="center">
    <i>Tasks</i> is a lean tasking system based on 
    <a href="https://github.com/ruby/rake">
    Ruby's Rake
    </a>
    <br><br>
    <img 
        alt="Arturo logo" 
        width="20" 
        src="https://github.com/arturo-lang/arturo/raw/master/docs/images/logo.png#gh-light-mode-only"
    />
    <img 
        alt="Arturo logo" 
        width="20" 
        src="https://github.com/arturo-lang/arturo/raw/master/docs/images/logo-lightgray.png#gh-dark-mode-only" 
    />
</p>

## At a Glance

<!-- <p align="center">
<img 
    alt="Running Tasks from terminal"
    width="720"
    src="./docs/running tasks screenshot.png"
/>
</p> -->

## Trying Tasks

*Tasks* may be splited into two sections: *runner* and *definitions*.

*The runner* is the section responsible to get the user input and then execute the defined tasks.
While *the definitions* are responsible to group the rules and logic of each task.


At your task file, you can do:

```art
import {tasks}!

; here goes the task definition

executeTask "my-task"
```

And then, runnning with:

```
arturo tasks.art
```

> [!TIP] 
> You may want to use a hashbang to don't need to call arturo for every run.
> Try: `#! arturo`

> [!TIP]
> To dynamically select the task via CLI, use:
> ```art
>  import {tasks}!
>  
>  ; tasks goes here
> 
>  ensure.that: "Argument needed for this script" 
>      -> not? empty? arg
>  runTests arg\0
> ```

### The *tasks' definition* itself

A real example of tasks:

```art
import {tasks}!

routine 'test [
    |> 'echo {Testing source code...}
]

task 'build .requires: [test] [
    source: ["./tests/example/example.c"]
    directory ./"tests/bin"

    file.as: 'f ./"tests/bin/example.exe" .requires: source [
        |> 'gcc print <= ~{|join.with: " " f\requires| -o |f\file|}
    ]
]

task 'run [
    file ./"tests/bin/example.exe" .as: 'f 
        -> execute.directly f\file
]

task 'clean [
    delete.directory ./"tests/bin"
]

executeTask 'build
executeTask 'run
executeTask 'clean
```

This will show you:

```

===========
>> BUILD


--------
* test

Testing source code...

./tests/example/example.c -o tests\bin\example.exe


=========
>> RUN

Hello, from Arturo's tasks
===========
>> CLEAN


``` 

## Documentation

### *Runner*
- `runTask: $[target :string :literal]`:
    Runs the `target` task

### *Task Definition*
- `directory: $[path :string]`:
    Creates a directory. The same as `write.directory path null`.
- `file: $[filename :string action :block]`:
    Defines a file. Files may require other files to exist. And also can be threated by other name using `.as`.
    - `.as :literal`:
        Wraps the file and its requirements into a dictionary: `#[file :string requires [:string]]`
    - `.requires`:
        The required files to this file exist. If none is passed, the files depends on itself.
- `exec $[command :string :literal params :string]`:
    Executes a command on shell.
    - alias: `|>`.
    - OBS.: This function is only available into `action`s from `routine`s and `task`s.
- `routine: $[name :string :literal, action :block]`:
    Defines a `:routine` that executes an `action`.
- `task: $[name :string :literal, action :block]`:
    Defines a `:task`, every task is a `:routine` but executable direclty from the `executeTask`.
    - `.requires [:literal :string :word]`:
        Defines the required routines to run this.
    - `.defers [:literal :string :word]`:
        Defines the required defer routines.


> [!WARNING]
> Never import this lib as `.lean`, or this will break the current code.
> This happens due to the nature of Arturo (being kind-of concatenative).

---