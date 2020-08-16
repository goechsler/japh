My old JAPH
===========

I recently came across an old Perl program on an old hard drive. There's a story to it.

I used to be a Perl hacker. In 2008 I started working for an ISP in Munich as a web programmer, where we mainly used Perl to build quite awesome web applications.

Even back then Perl was considered to be the old school language of web programming. Ruby was coming on rails, PHP was basically crap, but already widely used and Python was way to cool for school.
But in the context of an ISP Perl was certainly still the language of choice for many jobs.

So Perl it was, "enlightened" Perl to be precise. The term was coined to separate the community from the history of being a "write-only" scripting language, literally being only one character away from peril.

Using it the englightened way with a modern MVC framework, test-driven development thanks to dependency injection and a "postmodern" object sytem was really up to snuff, even by todays standards.

But under the hood it was plain old perilous Perl. And no one was stopping you from doing something like this:

        @S=qw(2uF7t juvyr 6PF5 cevag fuvsg fcyvg Pq6 Z5r fho beq Z5rO6O
        100 39 10 pue P7YZ5r bpg);$S=q[i _{j(e@|)-m}@|=f//,'<a(+g`p(+;]
        .q"k(c\\h';b(@|){$?=&_;d o q$?/n+&_*n,o q&_+$?%n*l}";for(0..$#S
        ){$s=chr(97+$_);$S=~s/\b$s\b/$S[$_]/g;}$S=~y/a-z/n-za-m/;eval$S

There it is. My JAPH, a few lines of code that rather look like random noise than like a computer program. Go ahead and look up JAPH on Wikipedia, if you're not familar with it. It's quite a throwback to the eighties.

If you would execute the code on a shell, it should print "Just another Perl hacker" and exit. The fact that this is not at all obvious is part of the fun.

I used to have this as a nerdy email signature for a while. You know, back when email was still a thing. And when people actually knew what the hell an email signature is.

How it does what it does
------------------------

So it writes "Just another Perl hacker", but how? I'm writing this over ten years after I made it, so even if I still roughly know what I did, I'm really excited to find out myself.

Let's take this apart. We've got a list @S:

         @S = qw(2uF7t juvyr 6PF5 cevag fuvsg fcyvg Pq6 Z5r fho beq Z5rO6O 100 39 10 pue P7YZ5r bpg);

And a string $S:

        $S = q[i _{j(e@|)-m}@|=f//,'<a(+g`p(+;].q"k(c\\h';b(@|){$?=&_;d o q$?/n+&_*n,o q&_+$?%n*l}";

There's a loop over the indices of @S:

        for (0..$#S) {
            $s = chr(97+$_);
            $S =~ s/\b$s\b/$S[$_]/g;
       }

There's a transliteration going on:

       $S =~ y/a-z/n-za-m/;

And then there's an eval of $S to make it do something:

       eval $S

Looks like the only interesting part is the loop. Let's have a closer look.
We're iterating over the indices of the list @S, which has 17 elements.
So we're actually doing:

       for (0 .. 16) { ... }

In the first iteration of the loop $_ is 0, so

       $s = chr(97 + 0);

Looking up character 97 in an ASCII table it turns out $s is set to the letter 'a'. (Be honest now, did you really know that or do you just think you remember it, like I do?) The last loop iteration would be character 97 + 16 = 113, which is the letter 'q'.

Ok, we're iterating over the letters 'a' to 'q' and we're doing some kind of replacement operation in our string $S:

        $S = ~s/\b$s\b/$S[$_]/g;

with the search part $s being a letter between 'a' and 'q', replacing it with the $respective element of the list @S . Looking up \b I found out it's matching a word boundary: It matches everywhere between a \w word character and a \W non-word character. This word boundary thing makes sure we're not replacing characters in already replaced parts. (At least it looks like this was the intention here. It's certainly not perfect. The character sequence "Z5r" is suspiciously common in our list ... nevermind.)

So in our string we're first replacing "a" with "2uF7t", "b" with "juvyr", ... and so on. Once we're done $S looks like this:

        fho _{beq(fuvsg@|)-39}@|=fcyvg//,'<2uF7t(+Pq6`P7YZ5r(+;Z5rO6O(6PF5\Z5r';juvyr(@|){$?=&_;cevag pue bpg$?/10+&_*10,pue bpg&_+$?%10*100}

Next the transliteration operator replaces the characters 'a-z' with 'n-za-m' which is the Perl equivalent of a ROT13 cipher on lower case letters. We end up with something that looks a bit more familiar:

        sub _{ord(shift@|)-39}@|=split//,'<2hF7g(+Pd6`P7YZ5e(+;Z5eO6O(6PF5\Z5e';while(@|){$?=&_;print chr oct$?/10+&_*10,chr oct&_+$?%10*100}

Round 2. Fight!
-----------------

Hey! Looks like we found a program in a program just like a nested Matryoshka doll. We're off to round two. Starting with a formatting makeover:

        sub _ {
            ord(shift @|) - 39
        }
        @| = split //, '<2hF7g(+Pd6`P7YZ5e(+;Z5eO6O(6PF5\Z5e';
        while (@|) {
            $? = &_;
            print chr oct $?/10 + &_*10, chr oct&_ + $?%10*100
        }

We see the definition of a subroutine with the inconspicuous name "_":

        sub _ {
            ord(shift @|) - 39
        }

The subroutine appears to shift an element out of some global, yet unknown list @| and calculates the ASCII value of the shifted element, which somehow suggests the element in question is a single ASCII character.

It returns whatever value we get minus 39. We've got no clue why. (And while you're pondering: Think about side effects for second. Do you see the i in peril? Great power comes with great responsibility.)

Luckily the next thing is the definition of the list @|, we were just wondering about. It's just a weird string split into a list of single characters. We knew it!

        @| = split //, '<2hF7g(+Pd6`P7YZ5e(+;Z5eO6O(6PF5\Z5e';

Now we've got a loop as long as @| is true, i.e. there's still elements in @|:

        while (@|) { ... }

In the loop we buffer our first return value from _ in a scalar variable called $?:

        $? = &_;

And then ... things get crazy. One round of formatting makeover may not be enough:

       print chr oct $?/10 + &_*10, chr oct&_ + $?%10*100

It's a "print" statement. It prints the results of two "chr" operations:

       print chr( oct( $? / 10 + &_ * 10 ) ) . chr( oct( &_ + $? % 10 * 100 ) )

Those two "chr" functions just turn the results of the respective nested "oct" functions into characters. Let's have a look at the "oct" functions, which interpret its argument as an octal string and returns the corresponding value.

        oct( $? / 10 + &_ * 10 )
        oct( &_ + $? % 10 * 100 )

Argument of the first statement: We divide our buffered value by 10 and add the next value from our subroutine multiplied by 10. Reading the docs on the "oct" function it turns out "oct" ignores any "trailing non-digits, such as a decimal point". So even if we may end up with a fraction as argument, it's interpreted as octal integer number.

Second statement: We take the next value from our subroutine and add the first decimal place of our buffered value multiplied by 100.

Apparently we take tree elements from the list and mix the first and second value together, as well as the first and third value. So three characters are lumped together to create two characters.

Let's walk through this. In the first iteration, we have '<' which has ASCII code 60. We start of with $_ being 60 - 39 = 21. For the follwing "2" we end up with the value (50 - 39 =) 11 and the third element "h" is converted to (104 - 39) = 65.

So our arguments look like:

        21/10 + 11 * 10 = 2 + 110 = 112.
        65 + 21 % 10 * 100 = 65 + 1 * 100 = 165

Octal 116 in decimal is 78 which translates to 'J' and octal 165 is decimal 117 which represents the character 'u'. "<2h" is transformed to "Ju", "F7g" is translated to "st", and so on ...

Looking a bit closer, it's not all that magical. In the first statement we divide our buffered value by 10. As long as the value is smaller than 100, the result of the division will be smaller than 10. So it will only occupy one digit. And by multiplying the second element by 10 we exactly free up that one decimal place. The same idea shows up in the second statement: by multiplying the first decimal place of our buffered value by 100 we free up the two digits for the third value, as long as this third value is smaller than 100. As the weird string and the whole program was specifically crafted for the one purpose of printing "Just another Perl hacker" it's no suprise that these "< 100" conditions hold true.

What was I thinking?
--------------------

I only remember partly what the thought process was. I do remember fondly spending lots of time fiddling arround to find something that works. Especially getting the formatting right was quite tricky. You'll probably noticed that it's not totally code golfed to death, just in order to make it line up nicely. You see, it's not about efficiency. It's all about beauty, obfuscation and weirdness.

I guess the inner part of the Matryoshka code was inspired by base64 encoding. Transforming characters, but not using the character borders to do so, seemed very appealing to me. Instead of doing so on a bit level I went with some crazy arithmetic on ASCII codes, which in the end had somehow similar effects.

The outer part is surely inspired by Dean Edward's javascript packer. I remember finishing the inner part and thinking that it's not obscure enough. Any experienced Perl hacker would easily see what's going on. As the result of the program was a given anyway, I really wanted to make sure that the idea behind it could not be understood at a glance. The packer did just that as it was used for obfuscation of javascript and I already had hacked it to get hold of some code that has been garbled with it. It's no surprise that the packer got out of style quickly. It was neither saving bandwidth nor provided protection against anyone who would be able to understand enough javascript to be interested in the unobfuscated code in the first place. So it's clearly bogus, but: It's a brilliant piece of code and I stole the idea to hide my ways in the japh.

Sprinkle it with a bit of ROT13 cipher. Serve cold like revenge. Enjoy.
