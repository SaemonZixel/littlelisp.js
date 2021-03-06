# Little Lisp JS

A mini Lisp interpreter in JavaScript. Supports lists (obvs), function invocation, lambdas, OOP (!), TCO, Macroses (experimental), embedded JSON, if/while statements, numbers, strings and many other. Use "window" as default scope. Can call a JS functions, read/write object fields, call object methods, create JS objects.

* Original author: Mary Rose Cook <mary@maryrosecook.com>
* Fully refactored by Saemon Zixel <saemonzixel@gmail.com>
* Have IDE with Debugger (!), Class Browser and Object explorer.

## Examples

```lisp
(window.alert "ok!")
```

```lisp
(setq x "ok!") 
(window.alert x)
```

```lisp
(setq x "ok!") 
(window.alert (x.substring 0 2))
```

```lisp
(setq x 1) 
(if (= x 1)
	(alert "x == 1") 
:else 
	(alert "x != 1")
)
```

```lisp
;; examples of syntactic sugar (implemented in parser)
(x < 2)                ;; -> (< x 2)
((x < 2) && (x > -1))  ;; -> (&& (< x 2) (> x -1))
((x < 2) || (x > -1))  ;; -> (|| (< x 2) (> x -1))
(x + 1 2 3)            ;; -> (+ x 1 2 3) => x+1+2+3
(x ++)                 ;; -> (++ x) 
(++ x y z)             ;; -> increments variables x, y, z separately
```

```lisp
(setq x (window.prompt "Value of X:")) ;; enter any value
(if (x = null)
	(setq x "(none)")
	(alert (+ "x = " x))
:else
	(alert (+ "x = " x))
)
```

```lisp
(setq x (window.prompt "Value of X:")) 

;; empty string and 0 will be cast to false
(if x 
	(alert (+ "x = " x))
:elseif (x = " ")
	(alert "x = (empty string)")
:else
	(alert "x = (none)")
)
```	

```lisp
(setq i 0)
(while (i < 10)
	(console.log i)
	(++ i)
)
```

```lisp
(setq i -1)
(while ((++ i) < 10)
	(console.log i)
	(if (i = 5) (break) 
	:else (continue))
)
```

```lisp
(setq i 0 i2 0)
(while ((i < 10) && (i2 < 10))
	(console.log (i + i2))
	(++ i i2)
)
```

```lisp
(setq i 0 list (1 2 3 4 5))
(while (i < list.length)
	(console.log list.@i)
	(++ i)
)
```

```lisp
;; JS object creation example
(setq obj1 (new Object))
(setq obj1.a 123 obj1.b "abc")

(setq name "c")
(setq obj1.@name ())
(setq obj1.@name.0 111 obj1.@name.1 222 obj1.@name.2 333) 

(alert (obj1.c.join " ")) ;; show "111 222 333"
```

```lisp
((lambda (x y) (window.alert (x + y)) "a" "b")
```

```lisp
(defun f1 (x) (alert (x + " - ok!")))
(f1 "test 'f1'")
```

```lisp
(typeof true) ;; "boolean"
(typeof undefined) ;; "undefined"
(typeof 123) ;; "number"
(typeof 123.45) ;; "number"
(typeof "abc") ;; "string"
(typeof abc) ;; "undefined" - because it's variable name 
(typeof :abc) ;; "atom" - in JavaScript it's String object with "atom_prefix" field
(typeof :abc-def-123) ;; "atom"
(typeof 'abc) ;; "atom"
(typeof #abc) ;; "atom"
(typeof it_is.0nly.1.at-om) ;; "undefined"
(typeof (1 2 3)) ;; "object" - return [1, 2, 3]
(typeof '(abc cde fgh)) ;; "object" - in JavaScript array is object
(typeof (new Object)) ;; "object"
(typeof null) ;; "object" - it's  historical bug of JavaScript language
(typeof 123 :eq "number") ;; return true
```

```lisp
(catch (a.b)) ;; return Exception object (Error: "a" is undefined!)
(throw (new Error "Error!")) ;; throws JavaScript exception
```

```lisp
;; evalute JS code
((new Function "" "alert(document.title);")) ;; show title of opened web page
```

```lisp
;; embedded JSON
(setq obj1 {name: "object1", data:[1, 2, 3], toString: function(){ return this.name + "=" + this.data.join(","); }})
```


## OOP examples

```lisp
(defclass Class1
	:extends String
	:instvars "inst_var1 inst_var2"
	:constructor init
	:classvars "singleton_instance"
)

(defmeth-static Class1.getSingleton (arg1)(
	(if (Class1.singleton_instance = null)
		(setq Class1.singleton_instance (new Class1 arg1))
	)
	(return Class1.singleton_instance)
))

(defmeth Class1.alert (arg1) (
	(alert (this.inst_var1 + arg1))
))

;; will be constructor
(defmeth Class1.init (arg1) (
	(setq this.inst_var1 arg1)
	
	(if (arguments.length > 1)
		(setq arg_key 2)
		(while (arg_key < arguments.length)
			(put this arguments.@arg_key (get arguments (arg_key + 1)))
			(++ arg_key arg_key)
		)
	)
))

(setq tmp1 (new Class1 123 :inst_var2 "abc"))
(window.alert tmp1.inst_var1)
(tmp1.alert "456")
```

### Macroses support (EXPERIMENTAL) 
```lisp
(defmacro assign (var_name value) `(setq ,var_name ,value))
(assign tmp 123)
```

```lisp
(defmacro foreach (field_name _in obj _do code) 
	(setq tmp_keys (gensym) tmp_count (gensym) tmp_indx (gensym))
	
	`(let ((,tmp_keys (Object.keys ,obj)) (,tmp_count (get ,tmp_keys "length")) (,tmp_indx -1))
		(while ((++ ,tmp_indx) < ,tmp_count)
			(setq ,field_name (get ,tmp_keys ,tmp_indx))
			~code  ; Comma may be replaced to tilda
		)
	)
)

(foreach name :in document.location :do (console.log name))

```