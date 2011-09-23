#JSONCode Compiler 1.0
Specification for JSONCode compiler.
***

JSONCode is basically a set of special expressions embedded as 'keys' with the intention to manipulate and transform the document.

## Example 1:

    {
        "@return": 25
    }

Result:

    20

## Example 1.2:

    {
		"@set(variable1)": 25
        "@return": {
			"@get(variable1)": null
		}
    }

Result:

    20

## Example 2:

    {
	    "name": "Johan",
		"age": {
			"@return": 25
		}
    }

Result:

    {
	    "name": "Johan",
		"age": 25
    }

## Example 3:

    {
	    "name": "Johan",
		"age": 25,
		"fibonacci": {
			"@set(i)": 1,
			"@return": {
				"@while(i < 10)": {
					"@set(i)": {
						"@sum": [
							"@get(i)":null,
							"@literal": 1
						]
					}
					@"return": {
						"@get(i)": null
					}
				}
			}
		}
    }

Result:

    {
	    "name": "Johan",
		"age": 25,
		"fibonacci":[0,1,1,2,3,5,8,13,21]
    }

## Example 3.1:

    {
	    "name": "Johan",
		"age": 25,
		"fibonacci": {
			"@set(i)": 1,
			"@loop": {
				"@set(i)": {
					"@sum": [
						"@get(i)":null,
						"@literal": 1
					]
				},
				"@if(i < 21)": {
					"@break": null
				},
				"@return": {
					"@get(i)": null
				}
			}
			
		}
    }

Result:

    {
	    "name": "Johan",
		"age": 25,
		"fibonacci":[0,1,1,2,3,5,8,13,21]
    }

## Example 4:

    {
		"@Data.Create": {
			"collection": "contacts",
			"item": {
				"email": "some@example.com",
				"name": "John Doe",
				"home": true
			}
		},
	    "@Data.FindOne": {
			"collection": "contacts",
			"condition": {
				"home": "true"
			}
		},
		"@return": {
			"isAtHome":{
				"@bind(item.home)": null
			},
			"contact": {
				"@if(item == null)": {
					"@return": null
				},
				"@else": {
					"email": {
						"@bind(item.email)": null
					},
					"name": {
						"@bind(item.name)": null
					}
				}
			}
		}
    }

Result:

    {
	    "isAtHome": true,
		"contact": {
			"email": "some@example.com",
			"name": "John Doe"
		}
    }

# Expressions

In JSONCode any object can be converted to a Expression.

All the expressions begin with the at symbol `@`.

Example:

    {
	    "@return":50
    }

Result

    50

## Parts

Any expression has at least 3 parts.

* Key: Name of the expression.
* hint: (Optional) A random string that can be used as a simple hint for the expression to execute.
* Input: (Optional) Object used as arguments of the expression.

In the following example `set` is the name of the expression, `variable1` is the hint and `2` is the input.

    {
	    "@set(variable1)": 2
    }

The syntax would be like this:

    {
	    "@<expression-name>(<hint>)": <expression>
    }

## Expression blocks

When a expression-key is present in a object definition that object becomes an expression-block and no JSON regular keys are allowed anymore.

The following expression-block is illegal, it contains both expressions and and regular JSON keys in the same object definition:

    {
	    "@set(name)": "Name",
		"regularJSONNumber": 2000 // This is illegal, @set already converted this object into a expression-block.
    }

The code should be formatted as follows:

    {
	    "@set(name)": "Name",
		"@return": {
			"regularJSONNumber": 2000
		}
    }

And the result would be:

    {
        "regularJSONNumber": 2000	
    }

Only the last expression of the block is actually took in count as the result of the whole block.

Example:
    
    {
	    "@return": 2,
	    "@return": 5,
    }

The result:

    5

## Input Processing

Since the input can be any JSON value it can also become another JSONCode expression.

In the following example you can see how the input of the high level `@return` is actually the result of nested `@return` Example:

    {
	    "@return": {
	    	"@return": {
				"point": {
					"x": 22.4,
					"y": 45.8
				}
			}
		}
    }

The result:

    {
		"point": {
			"x": 22.4,
			"y": 45.8
		}
	}

Depending of how the expression works the input can be ignored or processed. Conditional expressions like `@if` and `@else` uses this technique.

Example:

    {
		"@set(chuckNorrisIsHere)": false,
		"@if(chuckNorrisIsHere)": "Chuck is here, You are dead!",
		"@else": "It's never here!, you are safe!"
    }

The result is:

    "It's never here!, you are safe!"

## Built-in expressions

### @return
Returns the input expression as a result. Typically is the last expression in a block.

### @get
Returns the variable name in the hint. If no hint is given it returns undefined.

### @set
Sets the input as the value of the variable name provided in the hint. If no hint is given it does nothing. Always returns the input.

### @if
Conditional expression, takes a hint as a JS expression and executes and return the result of the input if the condition is met.  If there is no hint, it uses `lastStatusFlag` to determinate if it should return the input or not.

### @else
Conditional expression, takes a hint as a JS expression and executes and return the result of the input if the condition IS NOT MET. If there is no hint, it uses `lastStatusFlag` to determinate if it should return the input or not.

## Loops

Loops are repetitive expressions that process the input many times return an array of results.

### @loop
Process the input forever or until it founds a @break statement. Returns an array with the results of every input processing.

### @while
Process the input as long as the expression given in the hint is true or until it founds a loop control statement statement. Returns an array with the results of every input processing.

### @each
Takes a hint as a variable name or uses the last result of the expression-block and Process the input per item found in the given array or object. Returns an array with the results of every input processing.

#### @break

Immediately stops the execution of the current or any other iteration. Any result in the current iteration will *NOT* be added to the loop.

### @skip

Immediately stops the execution of the current iteration.  Any result in the current iteration will *NOT* be added to the loop.

### @continue

Immediately stops the execution of the current iteration. Unless you specify an input, any result in the current iteration will be added to the loop.

### @finish

Immediately stops the execution of the current or any other iteration. Unless you specify an input, any result in the current iteration will be added to the loop.

## @finish and @continue vs @break and @skip

All the loops generates arrays as results, this behavior can not be changed. Every time the input is executed the results are added to the array unless `@undefined` is returned (`@null` can be added though).

@break and @finish perform the same thing, prevent that any sentence in the loop from being executed; except that @finish let the last result of the current iteration or any input to be added as result of the loop array.

Same with @continue and @skip, @skip ignores the results while @continue adds any result or input.


## lastStatusFlag
It's a `level-scope` internal boolean field being set every time an expression is executed, true when the previous expression took place or false when it didn't.

## level-scope and conditional expressions
level-scope refers to a set of artifacts that are not transferred to inner scopes. The perfect example is the flag called `lastStatusFlag`.

Example:

    {
		"@if(true)": "meh...", //lastStatusFlag is being set as true because the expression took place.
		"@else(true)": "meh..." //lastStatusFlah is being set as false because the expression didn't take place(@else expects the conditional value to be negative)
		"@if(true)": { //lastStatusFlag is being set as true
			@else: "Yes!", // This is a new level, lastStatusFlag before this expression was undefined and now is being set as true.
			@else: "Never used" // Id doesn't execute because the last expression took place.
		}
		@if: "Awesome" //Executes successfully because the last expression took place and we didn't provided any hint as condition.
    }

Result:

    "Awesome"

### Errors Table
    Error Code			Message
	XXX					XXX


### Replacement tokens for the Error Messages.

**PROP_NAME** Name of the property involved in the error.

**CONVERTER_NAME** is the name of the converter involved in the error.

**SOURCE_NAME** name of the definition source. Usually is the name of the *.endpoint.json* ile where the error occurred.

**MESSAGE_NAME** is the name of the message involved in the error.

## Credits
**Author**: Johan Hernandez. *johan@firebase.co*

**Date:** Sept 23, 2011