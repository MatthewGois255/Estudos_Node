The idea of asynchronous code is not making it execute line by line at once

At first, putting a 'setTimeout(() => {console.log("Hello World)}, 3000)' in the middle of the code would make we think it runs the task above, hold on for 3 seconds, print out the "Hello World" AND THEN runs the following line. However, it doesn't work like that

Since 'setTimeout()' is an async method, it execute at the same time the next lines of code. Insteed of one main 'stream', the code get two streams that flow in parallel to each other

~~~
syncTask 1
.
.
setTimeout()    syncTask3
.               .
.               .
.               syncTask4
~~~

As both syncTask3 and setTimeout (the async code) runs at the sime time and syncTask3 has no delay, if syncTask3 depends on a variable that will instill be declared in the async code, it will become 'undefined'

~~~ javascript
let variable
setTimeout(() => {
    variable = "banana"
}, 3000) // 3 seconds delay
console.log(variable) // returns undefined
~~~

In order to fix it and make 'console.log' wait the sync code, so to speak, we can use *callbacks*. Both console.log() and setTimeout() are inserted into two different functions. The function containing console.log is called back inside the setTimeout method, just after 'variable' is initialized. Then we call the function containing the async method firts, since it will call the second function later

~~~ javascript
function banana(callback) {
    setTimeout(() => {
        const variable = "banana"
        callback(variable) // printf() is executed here, not at the bottom
        console.log(typeof(callback))
    }, 3000)
}

function printf(variavel) {
    console.log(variavel)
}

banana(printf)
~~~

The logic is the same if the callback doesn't receive any value. The point is that it's executed only after the setTimeout

~~~javascript
function banana(callback) {
    setTimeout(() => {
        const variable = "banana"
        callback()
    }, 3000)
}

function printf() {
    console.log("hello")
}

banana(printf)
~~~

~~~
banana(
    .
    .   
    setTimeout(
        .
        // 3 seconds
        .
        const variable = "banana"
        .
        .
        printf(variable)
        .
        .
        console.log(variable)
    )
)
~~~

But using many callbacks can make your code confuse and hard to deal with (generating the *calback hell*)

<br>

Another way to solve this problem is using **PROMISES**


Promises are closely related to then() methods. They are objects that represent the fulfillment or failure of an asynchronous operation. They indicate that there's more to come

With them, we don't need both a callback for success and a callback for error within the function. Promises take up to two arguments, fulfilled and rejected cases, just like the 'then()' method itself (that also returns a promises, allowing us to chain calls)

The two arguments of then() are in fact the callbacks we inserted into the first function, i. e. the two arguments are functions. One of these two callbacks will be executed depending on the result of the preceding promise. If it was fulfilled, the onFulfilled param will be executed, as long as both are located in the same place

If the function in then() take some parameters, these will come from the values of fulfilled or failure

Look at this example, where the promise is receiving or not a christmas gift, and what happens depending on the results

~~~javascript
function santa() {
    return new Promise((fulfilled, failure) => {
        let waitForChristmas = 1225
        setTimeout((naughty) => {
            naughty = true

            if(!naughty) {
                const gift = 'a bike'
                console.log(`Kevin was a good boy! ... His gift will be ${gift}`)
                fulfilled(gift)
            }
            
            else {
                const coal = "Someone's been a naughty boy"
                failure(coal)
            }

        }, waitForChristmas)
    })
}

// clear the code by separating the individual functions and just calling them
function goodboy(gift) {
    console.log(`Kevin received ${gift} as christmas gift`)
}

function naughtyboy(coal) {
    console.log(coal)
}

santa()
    .then(goodboy, naughtyboy)

// same output
santa()
    .then(goodboy)
    .catch(naughtyboy) // same as .then(null, onFailure), only for the second param. It's a way of catching potential errors

santa()
    .catch(naughtyboy)
~~~