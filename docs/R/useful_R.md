#Some R functions and features that will help with exercises

Remember: most solutions to your R problems can be found on the internet

##TRUE and FALSE evaluate to 1 and 0

	 a <- c(23,34,45,56,67,78,89)
	 a > 30
	[1] FALSE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE
	sum(a > 30)
	[1] 6
	sum(a > 70)
	[1] 2
	
##colSums gives you the sum of the columns in your dataframe

	#in this example
	#make random vector and put it in a 4 x 25 matrix and find the sums of the columns of the datframe
	set.seed(123)
	b <- rnorm(100)
	c <- matrix(b, nrow = 25, ncol =4)
	colSums(c)
	[1] -0.8332575  2.5534350  0.2560259  7.0643875
	
##Regular expressions

R regular expressions allow you to pull patterns from text. See the documentation for specifics.  Here is a guide.

http://www.stat.berkeley.edu/~nolan/stat133/Fall05/lectures/RegEx.html

Example

	d <- c("one##two","three##four","five##six","seven##eight","nine##ten")
	r <- regexpr('##',d)
	#regmatches pulls out the strings that match
	regmatches(d,r)
	[1] "##" "##" "##" "##" "##"
	#use of the wildcard to make the search less specific
	r <- regexpr('##.*',d)
	regmatches(d,r)
	[1] "##two"   "##four"  "##six"   "##eight" "##ten"  
	#gsub does both actions at once and uses *grouping*
	gsub('##(.*)',"",d)
	[1] "one"   "three" "five"  "seven" "nine" 