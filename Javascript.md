## Javascript 

### Server-side javascript objects

* string
```
var serverString = ee.String('This is on the server.');
print('String on the server:', serverString);
```

* number
```
var serverNumber = ee.Number(Math.E);
print('e=', serverNumber);
```

* operation
```
var logE = serverNumber.log();
print('log(e)=', logE);
```

* sequence
```
var sequence = ee.List.sequence(1, 5);
print('Sequence:', sequence);

var value = sequence.get(2);
print('Value at index 2:', value);

print('No error:', ee.Number(value).add(3));
```

* dictionary
```
var dictionary = ee.Dictionary({
  e: Math.E,
  pi: Math.PI,
  phi: (1 + Math.sqrt(5)) / 2
});

print('Euler:', dictionary.get('e'));
print('Pi:', dictionary.get('pi'));
print('Golden ratio:', dictionary.get('phi'));

print('Keys: ', dictionary.keys());
```

* date and time
```
var date = ee.Date('2015-12-31');
print('Date:', date);

var now = Date.now();
print('Milliseconds since January 1, 1970', now);

var eeNow = ee.Date(now);
print('Now:', eeNow);

var aDate = ee.Date.fromYMD(2017, 1, 13);
print('aDate:', aDate);

var theDate = ee.Date.fromYMD({
  day: 13,
  month: 1,
  year: 2017
});
print('theDate:', theDate);
```

* apply function to each element in a list using __map__
```
var serverList = ee.List.sequence(0, 7);
serverList = serverList.map(function(n) {
  return ee.Number(n).add(1);
});
print(serverList);
```

* script modules
```
// define module and save in a js file
exports.doc = 'The Foo module is a demonstration of script modules.' +
    '\n It contains a foo function that returns a greeting string. ' +
    '\n It also contains a bar object representing the current date.' +
    '\n' +
    '\n foo(arg):' +
    '\n   @param {ee.String} arg The name to which the greeting should be addressed' +
    '\n   @return {ee.String} The complete greeting.' +
    '\n' +
    '\n bar:' +
    '\n   An ee.Date object containing the time at which the object was created.';
exports.foo = function(arg) {
  return 'Hello, ' + arg + '!  And a good day to you!';
};
exports.bar = ee.Date(Date.now());


// use the module by require
var Foo = require('users/username/default:Modules/FooModule.js');
print(Foo.doc);
print(Foo.foo('world'));
print('Time now:', Foo.bar);
```
