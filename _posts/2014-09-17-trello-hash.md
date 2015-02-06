---
layout: post
category : code
tagline: "solving Trello's developer challenge"
tags : [php, code, trello, hash]
---

Our team was looking into project / client management solutions the other day and I came across [Trello](http://www.trello.com). While looking around their site, I stumbled upon their [developer](https://trello.com/jobs/developer) job listing, which included a challenge to solve as part of the application process. It looked pretty fun, so I tackled it when I got home. 

## The Challenge

The challenge, in a nutshell, is to reverse engineer a sample hash function. The give pseudocode for the function is as follows:

{% highlight php startinline linenos %}
Int64 hash (String s) {
    Int64 h = 7
    String letters = "acdegilmnoprstuw"
    for(Int32 i = 0; i < s.length; i++) {
        h = (h * 37 + letters.indexOf(s[i]))
    }
    return h
}
{% endhighlight %}

The function should work as follows: `hash("leepadg")` returns `680131659347`. The question is, what string returns `956446786872726`?

## My solution

To really get an understanding of the pseudocode, I decided to implement it in `php`. Essentially a one-to-one translation is the following:

{% highlight php startinline linenos %}
function hash2($s)
{
	$h = 7;

	$letters = "acdegilmnoprstuw";

	for ($i=0; $i < strlen($s); $i++) { 
		$pos = strpos($letters, $s[$i]);
		if ($pos === false) {
			$char = $s[$i];
			unset($pos);
			throw new ErrorException("The character {$char} does not exist in the string {$letters}");
		} else {
			$h = ($h * 37 + $pos);
		}

	}
	return $h;
}
{% endhighlight %}

### Reimplementing the function recursively

While going through this, it seemed to me that this would be pretty easy to implement recursively, and I definitely felt that to reverse the hash, I'd have to do it recursively. So, I rewrite the hash function using recursion. To do this, we need to also know how deep to recurse (so that we don't create an infinite recursion loop). So, as a second parameter, we are going to pass `strlen($s)`.

{% highlight php startinline linenos %}
function recursive_hash2($s, $j)
{
	$letters = "acdegilmnoprstuw";

	if ($j == 0) {
		return 7;
	} else {
		return recursive_hash2($s, --$j) * 37 + strpos($letters, $s[$j]);
	}
}
{% endhighlight %}

### Reversing the function

So, in thinking about reversing the hash, I knew that we'd be able to at least get the last letter of the final string (ie, the `strpos($letters, $s[$j])`) by `$number % 37`. Then, we could reduce the number by reversing the math with `$number - ($number % 37) / 37`. I tested this out manually, and verified that it worked, and knew this result is what we would recurse on. But, we'd need to kill recursion at some point, right? That math would just go into negative numbers. So, we'd need a test to kill the recursion. That led me to this final solution:

{% highlight php startinline linenos %}	
function reverse_hash2($number)
{
	$letters = "acdegilmnoprstuw";

	$reduced = ($number - ($number % 37)) / 37;
	if ($number < 16) {
		return null;
	} else {
	 	return reverse_hash2($reduced) . $letters[$number%37];
	}	
}
{% endhighlight %}

Notice that we need to concatenate the string as the nested functions exit.

---

I've got the full code available on [GitHub](https://github.com/oflannabhra/trello-app). Thanks to Trello for a fun challenge!!