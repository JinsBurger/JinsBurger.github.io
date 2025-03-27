---
layout: post
title:  "Running x86_64 Ubuntu on Apple Silicon (M-Series) MacBooks"
date:   2023-03-27 09:00:00 +0900
categories:
---

# Syntax highlighting
This theme supports syntax respectively code highlighting. Below you find some examples of different programming languages.

Sed ut perspiciatis unde omnis iste natus error sit voluptatem accusantium doloremque laudantium, totam rem aperiam, eaque ipsa quae ab illo inventore veritatis et quasi architecto beatae vitae dicta sunt explicabo. Nemo enim ipsam voluptatem quia voluptas sit aspernatur aut odit aut fugit, sed quia consequuntur magni dolores eos qui ratione voluptatem sequi nesciunt. Neque porro quisquam est, qui dolorem ipsum quia dolor sit amet, consectetur, adipisci velit, sed quia non numquam eius modi tempora incidunt ut labore et dolore magnam aliquam quaerat voluptatem. Ut enim ad minima veniam, quis nostrum exercitationem ullam corporis suscipit laboriosam, nisi ut aliquid ex ea commodi consequatur? Quis autem vel eum iure reprehenderit qui in ea voluptate velit esse quam nihil molestiae consequatur, vel illum qui dolorem eum fugiat quo voluptas nulla pariatur?

{% highlight rust %}
// 암호화 함수
fn encrypt(text: &str, shift: i16) -> String {
    // 'A'와'Z'의 문자코드를 i16 타입으로 취득 --- (*1)
    let code_a = 'A' as i16;
    let code_z = 'Z' as i16;
    // 결과를 대입할 변수를 선언
    let mut result = String::new();
    // 한 글자씩 치환 처리 --- (*2)
    for ch in text.chars() {
        // 문자코드를 변환
        let mut code = ch as i16;
        // A와Z 사이에 있는 값인가？ --- (*3)
        if code_a <= code && code <= code_z {
            // shift만큼 뒤의 문자로 치환 --- (*4)
            code = (code - code_a + shift + 26) % 26 + code_a;
        }
        // 문자코드를 다시 문자로 변환 --- (*5)
        result.push((code as u8) as char);
    }
    return result;
}

fn main() {
    // 함수 호출
    let enc = encrypt("I LOVE RUST.", 3);
    let dec = encrypt(&enc, -3);
    println!("{} => {}", enc, dec);
}
{% endhighlight %}

Sed ut perspiciatis unde omnis iste natus error sit voluptatem accusantium doloremque laudantium, totam rem aperiam, eaque ipsa quae ab illo inventore veritatis et quasi architecto beatae vitae dicta sunt explicabo. Nemo enim ipsam voluptatem quia voluptas sit aspernatur aut odit aut fugit, sed quia consequuntur magni dolores eos qui ratione voluptatem sequi nesciunt. Neque porro quisquam est, qui dolorem ipsum quia dolor sit amet, consectetur, adipisci velit, sed quia non numquam eius modi tempora incidunt ut labore et dolore magnam aliquam quaerat voluptatem. Ut enim ad minima veniam, quis nostrum exercitationem ullam corporis suscipit laboriosam, nisi ut aliquid ex ea commodi consequatur? Quis autem vel eum iure reprehenderit qui in ea voluptate velit esse quam nihil molestiae consequatur, vel illum qui dolorem eum fugiat quo voluptas nulla pariatur?

{% highlight solidity %}
pragma solidity ^0.8.7;

import "@openzeppelin/contracts/utils/Address.sol";

contract Staking {
    using Address for address;

    // Parameters
    uint128 public constant VALIDATOR_THRESHOLD = 1 ether;

    // Properties
    address[] public _validators;

    mapping(address => bool) public _addressToIsValidator;
    mapping(address => uint256) public _addressToStakedAmount;
    mapping(address => uint256) public _addressToValidatorIndex;
    uint256 public _stakedAmount;
    uint256 public _minimumNumValidators;
    uint256 public _maximumNumValidators;

    mapping(address => bytes) public _addressToBLSPublicKey;

  function validators() public view returns (address[] memory) {
        return _validators;
    }

    function validatorBLSPublicKeys() public view returns (bytes[] memory) {
        bytes[] memory keys = new bytes[](_validators.length);

        for (uint256 i = 0; i < _validators.length; i++) {
            keys[i] = _addressToBLSPublicKey[_validators[i]];
        }

        return keys;
    }

}
{% endhighlight %}
Sed ut perspiciatis unde omnis iste natus error sit voluptatem accusantium doloremque laudantium, totam rem aperiam, eaque ipsa quae ab illo inventore veritatis et quasi architecto beatae vitae dicta sunt explicabo. Nemo enim ipsam voluptatem quia voluptas sit aspernatur aut odit aut fugit, sed quia consequuntur magni dolores eos qui ratione voluptatem sequi nesciunt. Neque porro quisquam est, qui dolorem ipsum quia dolor sit amet, consectetur, adipisci velit, sed quia non numquam eius modi tempora incidunt ut labore et dolore magnam aliquam quaerat voluptatem. Ut enim ad minima veniam, quis nostrum exercitationem ullam corporis suscipit laboriosam, nisi ut aliquid ex ea commodi consequatur? Quis autem vel eum iure reprehenderit qui in ea voluptate velit esse quam nihil molestiae consequatur, vel illum qui dolorem eum fugiat quo voluptas nulla pariatur?


<br />ruby:
{% highlight ruby %}
def foo
  puts 'foo'
end

def bubble_sort(list)
  return list if list.size <= 1 # already sorted
  swapped = true
  while swapped do
    swapped = false
    0.upto(list.size-2) do |i|
      if list[i] > list[i+1]
        list[i], list[i+1] = list[i+1], list[i] # swap values
        swapped = true
      end
    end
  end

  list
end
{% endhighlight %}


<br /><br />python:
{% highlight python %}
def func():
     # function body
     print("hello world!")

     def setup(app):
         # enable Pygments json lexer
         try:
             import pygments
             if pygments.__version__ >= '1.5':
                 # use JSON lexer included in recent versions of Pygments
                 from pygments.lexers import JsonLexer
             else:
                 # use JSON lexer from pygments-json if installed
                 from pygson.json_lexer import JSONLexer as JsonLexer
         except ImportError:
             pass  # not fatal if we have old (or no) Pygments and no pygments-json
         else:
             app.add_lexer('json', JsonLexer())

         return {"parallel_read_safe": True}

words = ['cat', 'window', 'defenestrate']
for w in words:
   print w, len(w)
{% endhighlight %}


<br /><br />php:
{% highlight php %}
<?php function add($x, $y) {
    $total = $x + $y;
    return $total;
}
echo "1 + 16 = " . add(1, 16);
?>
{% endhighlight %}



<br /><br />js:
{% highlight javascript %}
function sayHello(name) {
  if (!name) {
    console.log('Hello World');
  } else {
    console.log(`Hello ${name}`);
  }  
}  

function myFunc(a, b) {
    return a * b;
}
document.getElementById('demo').innerHTML = myFunc(4, 3);
{% endhighlight %}


<br /><br />java:
{% highlight java %}
class HelloWorldApp {
    public static void main(String[] args) {
        System.out.println("Hello World!");
    }
}
{% endhighlight %}


<br />objective c:
{% highlight objective_c %}
- (int)method:(int)i {
    return [self square_root:i];
}
{% endhighlight %}


<br /><br />perl:
{% highlight perl %}
while (<>) {
    chomp;
    if (s/$//) {
        $_ .= <>;
        redo unless eof();
    }
}
{% endhighlight %}


<br /><br />sql:
{% highlight sql %}
SELECT Country FROM Customers WHERE Country <> 'USA'
{% endhighlight %}


<br /><br />c++:
{% highlight c++ %}
#include
using namespace std;
int main () {
  cout << "Hello World!";
  return 0;
}
{% endhighlight %}


<br /><br />c sharp:
{% highlight c# %}
class Foo {
    public int Value;
    public static explicit operator Foo(int value) {
        return new Foo(value);
    }
}
Foo foo = (Foo)2;
{% endhighlight %}


<br /><br />vb:
{% highlight vb linenos %}
Private Sub Form_Load()
    MsgBox "Hello, World!"
End Sub
{% endhighlight %}
