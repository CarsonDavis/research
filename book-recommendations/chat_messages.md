# Setup and Initial Exploration

## Message 1
> let's make a folder for book-recommendations. go to my downloads and find the latest goodreads import. then use python to make sure you understand the file. write a quick guide in markdown on how to read it. we care specficaly about the title of the book, the rating I personally gave it, and the review that I left. we only care about books that i have marked as read, and nothing else

## Message 2
> ok, let's make a csv with just the key metadata. title, author, my rating, my review, and the goodreads identification. make sure and double check that you can use that id to get to the goodreads website for that book

## Message 3
> ok, when you found the goodreads page, did it have the genre info

# Build Goodreads Module

## Message 4
> perfect. let's make a clean module inside this folder that has two utilities. one to turn a raw export into that clean export and one that scrapes the genres from goodreads and adds them. it should be able to handle failures and retries, and have a progress indicator. it would also be good to have some simple parallelization using modern 2025 python techniques.

## Message 5
> great. let's populate the whole thing.

## Message 6
> which ones are missing genres?

## Message 7
> ah ok. well, do a google search and update their genres manually

# Refinements

## Message 8
> i notice the progress bar has every parallel process displaying a total of 581, as if each one is working on the entire stack. double check our code.

## Message 9
> do we need the main at the root level

## Message 10
> where is the original goodreads export? lets move all the goodreads files into a subfolder and update our scripts to match

# Re-run with Correct Export

## Message 11
> we are using the wrong goodreads file? it should have been from today and have 900+ lines. find it and let's re run everything

## Message 12
> check this one for missing genres too, and populate as before, either with the old values or with newly searched ones
