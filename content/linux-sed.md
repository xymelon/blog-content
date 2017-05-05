Title: Linux学习笔记——'Pattern Space' and 'Hold Space' in sed
Date: 2013-12-02 20:01
Category: Technology
Tags: Linux
Slug: linux-sed-space
Author: xymelon

来源：[http://stackoverflow.com/questions/12833714/the-concept-of-hold-space-and-pattern-space-in-sed](http://stackoverflow.com/questions/12833714/the-concept-of-hold-space-and-pattern-space-in-sed)

When sed reads a file line by line, the line that has been currently read is inserted into the pattern buffer (pattern space). Pattern buffer is like the temporary buffer, the scratchpad where the current information is stored. When you tell sed to print, it prints the pattern buffer.

Hold buffer / hold space is like a long-term storage, such that you can catch something, store it and reuse it later when sed is processing another line. You do not directly process the hold space, instead, you need to copy it or append to the pattern space if you want to do something with it. For example, the print command p prints the pattern space only. Likewise, s operates on the pattern space.

<!-- PELICAN_END_SUMMARY -->

Here is an example:

    sed -n '1!G;h;$p'

(the `-n` option suppresses automatic printing of lines)

There are three commands here: `1!G`, `h` and `$p`. `1!G` has an address, 1 (first line), but the `!` means that the command will be executed everywhere but on the first line. `$p` on the other hand will only be executed on the last line. So what happens is this:


> 1. first line is read and inserted automatically into the pattern space
> 1. on the first line, first command is not executed; `h` copies the first line into the hold space.
> 1. now the second line replaces whatever was in the pattern space
> 1. on the second line, first we execute `G`, appending the contents of the hold buffer to the pattern buffer, separating it by a newline. The pattern space now contains the second line, a newline, and the first line.
> 1. Then, `h` command inserts the concatenated contents of the pattern buffer into the hold space, which now holds the reversed lines two and one.
> 1. We proceed to line number three -- go to the point (3) above.


Finally, after the last line has been read and the hold space (containing all the previous lines in a reverse order) have been appended to the pattern space, pattern space is printed with `p`. As you have guessed, the above does exactly what the tac command does -- prints the file in reverse.
