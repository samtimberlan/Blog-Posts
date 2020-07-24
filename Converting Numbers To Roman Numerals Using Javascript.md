## The Challenge:

When converting numbers to Roman numerals, there are certain restrictions. A particular figure is not to appear more than thrice. This is done to enhance readability and reduce confusion. For example, IIII might be mistaken for III. Consequently, When a symbol appears after a larger (or equal) symbol it is added. But, if the symbol appears before a larger symbol it is subtracted.

## The Solution:

There are several approaches and algorithms to solve this challenge. However, the algorithm a friend of mine and I came up with is outlined below:

- Declare a nested array. This contains symbols for special Roman numerals. Each element of the parent array holds symbols for units, tens, hundreds and thousands. Furthermore, include symbols for special numerals like four and nine. This would save you lines of code.

```
var RomanNumerals = [
  ["I", "IV", "V", "IX"],      //Units
  ["X", "XL", "L", "XC"],      //Tens
  ["C", "CD", "D", "CM"],      //Hundreds
  ["M"]                        //Thousands(In this case only M. Roman numerals don't exceed 3999)
];
```

- Create a reusable function to perform the numeric conversion. It accepts two parameters, the number to be converted and an array indicating the value of the number – unit, tens, hundreds or thousands. The number to be converted is checked and assigned a value from the array, corresponding to its Roman equivalent. Here’s a sneak peek into the function.

```
function convertToDigitRoman(num, romArr) {
  var arrayToBeUsed = romArr;
  if (num <= 3) {
    while (num > 0) {
      romanNumResultArr.push(arrayToBeUsed[0]);
      num--;
    }
  }
  if (num == 4) {
    romanNumResultArr.push(arrayToBeUsed[1]);
  }

  if (num >= 5 && num < 9) {
    romanNumResultArr.push(arrayToBeUsed[2]);
    while (num - 5 > 0) {
      romanNumResultArr.push(arrayToBeUsed[0]);
      num--;
    }
  }

  if (num == 9) {
    romanNumResultArr.push(arrayToBeUsed[1]);
  }
}
```

- Now we have a reusable function converting digits, it’s time to use this function to convert numbers with multiple digits like 10 or 265. To achieve this, we use another function. It breaks the number into an array of single numbers. Hence, 265 becomes `[2,6,5]`. Finally, we loop through each of them calling our reusable function to assign the proper roman numeral.
  Here’s the full code:

```
var RomanNumerals = [
  ["I", "IV", "V", "IX"],
  ["X", "XL", "L", "XC"],
  ["C", "CD", "D", "CM"],
  ["M"]
];
var romanNumResultArr = [];

function convertToRoman(num) {
//Roman numbers can't exceed 3999
  if (num >= 4000) {
    return "Unfortunately, numbers from 4000 and above do not have roman numerals";
  }

//Break the number into an array of numbers
  var givenNumeral = num
    .toString()
    .split("")
    .map(Number);


//The length of the givenNumeral tells us the value of the number - 2 would mean ten and 3, hundred.
  for (i = givenNumeral.length; i > 0; i--) {
    convertToRomanUnit(givenNumeral.shift(), RomanNumerals[i - 1]);
  }

  var result = romanNumResultArr.join("");

  console.log(result);
  return result;
}


//---------------Reusable Function-------------------------------

function convertToRomanUnit(num, romArr) {
  var arrayToBeUsed = romArr;
  if (num <= 3) {
    while (num > 0) {
      romanNumResultArr.push(arrayToBeUsed[0]);
      num--;
    }
  }
  if (num == 4) {
    romanNumResultArr.push(arrayToBeUsed[1]);
  }

  if (num >= 5 && num < 9) {
    romanNumResultArr.push(arrayToBeUsed[2]);
    while (num - 5 > 0) {
      romanNumResultArr.push(arrayToBeUsed[0]);
      num--;
    }
  }

  if (num == 9) {
    romanNumResultArr.push(arrayToBeUsed[1]);
  }
}
```

You can [view the full source code here](https://github.com/samtimberlan/Roman-Numeral-Converter/archive/master.zip).

There you have it. Thanks for reading.

Did you spot a typo, an error or want to contribute? [Here's the repo on GitHub](https://github.com/samtimberlan/Blog-Posts/blob/master/Converting%20Numbers%20To%20Roman%20Numerals%20Using%20Javascript.md)
