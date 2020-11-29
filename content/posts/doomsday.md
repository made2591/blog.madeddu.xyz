---
layout: post
title: "The Doomsday rule"
date: 2017-12-23
polly: https://madeddu.xyz/mp3/doomsday.mp3
categories:
- coding
- miscellaneous
- algorithms
tags:
- coding
- doomsday
- algorithm
- ruby
katex: true
markup: "mmark"
---

### The Doomsday rule
A few months ago I came across the name of J. H. Conway: you're wondering who the hell he is. Well, Conway is an English mathematician active in the theory of _finite groups_, _knot theory_, _number theory_, _combinatorial game theory_ and _coding theory_. He has also contributed to many branches of _recreational mathematics_ and he is the invention of the Game of Life. Ah, I was forgetting one last thing: he is currently __Professor Emeritus of Mathematics at Princeton University__ in New Jersey[^wiki]. Ok. let's respect this guy but...what would I talk to you about? Well, in this article I will talk about a magic trick: the Doomsday rule.

<div class="img_container"><img src="https://i.imgur.com/7W3ZIn2.jpg" style="width: 100%; marker-top: -10px;"/></div>

### A brief history
Who has ever heard of it? No ones? Ok, the first thing to know is that the Doomsday rule is an _algorithm_ of determination of the day of the week for a given date. This f****ng unbelievable trick of mind will allow you to win so many bets during the Christmas holidays that you will be envied by your nephews, close friends, neighbors, lovers and even your pets.

The mental calculation was devised by J. H. Conway in 1973: I don't know how much a _mentalist_ you have to be to trust this method, but the algorithm actually returns the correct answer: if you want to discover more about how it works, go ahead. If you want to try to discover how the algorithm works starting from the code, [here](https://github.com/made2591/doomsday/blob/master/doomsday.rb) my repo with a Ruby implementation of the algorithm.

### The algorithm
First of all, a big picture of how the algorithm works in three steps:
- __step 1__: determination of the anchor day for the century,
- __step 2__: calculation of the _doomsday_ for the year from the anchor day,
- __step 3__: selection of the closest date out of those that always fall on the doomsday, e.g., 4/4 and 6/6, and count of the number of days (modulo 7) between that date and the date in question to arrive at the day of the week;

What?! Do you understand something?! Because if not, you really have to know that...Conway can usually give the correct answer to question like "what day of week was the 13 February 1867?", in under two seconds[^joke]. If you're thinking that probably this is the main reason why you're reading this blog and he's __Professor Emeritus of Mathematics at Princeton University__, well, you're right.

In any case, since I'm a computer scientist, and I have absolutely no desire to count things in mind nor to open a calendar or search on google for my _quiz-date_, I have implemented the Conway algorithm for the calculation of Doomsday using Ruby.

### Preliminary explanation
This algorithm involves treating days of the week like numbers _modulo 7_: Conway often suggests - in his course in Princeton called _"How I will literaly blow your mind"_ - thinking of the days of the week as "Noneday"; or as "Sansday" (for Sunday), "Oneday", (for Monday, it sounds similar), "Twosday", "Treblesday", "Foursday", "Fiveday", and "Six-a-day". It doesn't matter, because there are tables with - wait for it - _Doomsdays for the Gregorian calendar_, or what I call "The __Year__ Doomsday". Like the one below:

| Mon. | Tue. | Wed. | Thu. | Fri. | Sat. | Sun. | Mon. | Tue. | Wed. | Thu. | Fri. | Sat. | Sun. |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |  |
| 1898 | 1899 | 1900 | 1901 | 1902 | 1903 | →    | 1904 | 1905 | 1906 | 1907 | →    | 1908 | 1909 |
| 1910 | 1911 | →    | 1912 | 1913 | 1914 | 1915 | →    | 1916 | 1917 | 1918 | 1919 | →    | 1920 |
| 1921 | 1922 | 1923 | →    | 1924 | 1925 | 1926 | 1927 | →    | 1928 | 1929 | 1930 | 1931 | →    |
| 1932 | 1933 | 1934 | 1935 | →    | 1936 | 1937 | 1938 | 1939 | →    | 1940 | 1941 | 1942 | 1943 |
| →    | 1944 | 1945 | 1946 | 1947 | →    | 1948 | 1949 | 1950 | 1951 | →    | 1952 | 1953 | 1954 |
| 1955 | →    | 1956 | 1957 | 1958 | 1959 | →    | 1960 | 1961 | 1962 | 1963 | →    | 1964 | 1965 |
| 1966 | 1967 | →    | 1968 | 1969 | 1970 | 1971 | →    | 1972 | 1973 | 1974 | 1975 | →    | 1976 |
| 1977 | 1978 | 1979 | →    | 1980 | 1981 | 1982 | 1983 | →    | 1984 | 1985 | 1986 | 1987 | →    |
| 1988 | 1989 | 1990 | 1991 | →    | 1992 | 1993 | 1994 | 1995 | →    | 1996 | 1997 | 1998 | 1999 |
| →    | 2000 | 2001 | 2002 | 2003 | →    | 2004 | 2005 | 2006 | 2007 | →    | 2008 | 2009 | 2010 |
| 2011 | →    | 2012 | 2013 | 2014 | 2015 | →    | 2016 | 2017 | 2018 | 2019 | →    | 2020 | 2021 |
| 2022 | 2023 | →    | 2024 | 2025 | 2026 | 2027 | →    | 2028 | 2029 | 2030 | 2031 | →    | 2032 |
| 2033 | 2034 | 2035 | →    | 2036 | 2037 | 2038 | 2039 | →    | 2040 | 2041 | 2042 | 2043 | →    |
| 2044 | 2045 | 2046 | 2047 | →    | 2048 | 2049 | 2050 | 2051 | →    | 2052 | 2053 | 2054 | 2055 |
| →    | 2056 | 2057 | 2058 | 2059 | →    | 2060 | 2061 | 2062 | 2063 | →    | 2064 | 2065 | 2066 |
| 2067 | →    | 2068 | 2069 | 2070 | 2071 | →    | 2072 | 2073 | 2074 | 2075 | →    | 2076 | 2077 |
| 2078 | 2079 | →    | 2080 | 2081 | 2082 | 2083 | →    | 2084 | 2085 | 2086 | 2087 | →    | 2088 |
| 2089 | 2090 | 2091 | →    | 2092 | 2093 | 2094 | 2095 | →    | 2096 | 2097 | 2098 | 2099 | 2100 |

So...for a day, be a doomsday is a properties - at least - related to years. For instance, doomsday for the current year in the Gregorian calendar (2017) is Tuesday. The Conway algorithm works because it is possible to _easily_ find the day of the week of a given calendar date by using a _nearby_ doomsday as a reference point. To help with this, Conway findout that there is a list of _easy-to-remember_ dates...for each month...

that always land on the doomsday of the given year: O.M.G.

What does it means? It means that, for instance, the last day of February defines a doomsday. Or January, January 3 is a doomsday during common years and January 4 a doomsday during leap years, and so on. What the hell of magic days are these doomsday?!? We will explain _how_ the Doomsday for a year is computed. For now, let be the following table a month table that correspond __always__ to the doomsday.

| Month     | Doomsdays                                     |
| --------- | --------------------------------------------- |
| January   | 3, 31 (solar year) 4 (leap years)             |
| February  | 7, 14, 21, 28 (solar year) 1, 29 (leap years) |
| March     | 7, 14, 21, 28                                 |
| April     | 4                                             |
| May       | 9                                             |
| June      | 6                                             |
| July      | 11                                            |
| August    | 8                                             |
| September | 5                                             |
| October   | 10                                            |
| November  | 7                                             |
| December  | 12                                            |

### Step 1 aka finding a year's base day
First of all, ask for a day

{{< highlight ruby >}}
print 'Insert day:   '
day   = gets.to_i
print 'Insert month: '
month = gets.to_i
print 'Insert year:  '
year  = gets
{{< / highlight >}}

Then, given

$$c = year's \; century + 1 $$

(for instance, \\(c(2017) = 20\\)), then compute:

$$\Big\{\Big[\Big(5c + \left \lfloor{\frac{c - 1}{4}}\right \rfloor\Big) \; mod \; 7\Big] + 4\Big\} \; mod \; 7$$

My really unefficient implementation of the formula using Ruby is:

{{< highlight ruby >}}
baseDay = (((5 * (year[0, 2].to_i + 1)) + (year[0, 2].to_i / 4).ceil % 7) + 4) % 7
puts baseDay
{{< / highlight >}}

For example, the base day for the 21st century is Tuesday, because:

$$\Big\{\Big[\Big(5 x 21 + \left \lfloor{\frac{21 - 1}{4}}\right \rfloor\Big) \; mod \; 7\Big] + 4\Big\} \; mod \; 7 = 2 = \; Tuesday$$

### Step 2 aka finding a year's Doomsday
Let be:

$$y = year's \; last \; 2 \; numbers $$

(for instance, \\(y(2017) = 17\\)), to determine the Doomsday of the year compute:

$$\Big[\Big(\left \lfloor{\frac{y}{12}}\right \rfloor + y \; mod \; 7 + \left \lfloor{\frac{y \; mod \; 12 }{4}}\right \rfloor \Big) \; mod \; 7\Big] + year's \; baseday = \; year's \; Doomsday$$

A possible implementation of the formula using Ruby is:

{{< highlight ruby >}}
doomsday = (((year[2, 4].to_i / 12).ceil + (year[2, 4].to_i % 12) + ((year[2, 4].to_i % 12) / 4).ceil) % 7) + baseDay
puts doomsday
{{< / highlight >}}

For example, the Doomsday of the year 1966 is Monday, because:

$$\Big[\Big(\left \lfloor{\frac{66}{12}}\right \rfloor + 66 \; mod \; 7 + \left \lfloor{\frac{66 \; mod \; 12 }{4}}\right \rfloor \Big) \; mod \; 7\Big] + 3 = 8$$

In _2010_, a __simpler method__ - remember, less than two seconds - was discovered to find the year's Doomsday. It has been shown that this method, more formaly called _"Odd + 11"_, is equivalent in formulas to compute:

$$\Big[\Big(y + \left \lfloor{\frac{y}{4}}\right \rfloor \Big) \; mod \; 7 \Big] + year's \; baseday = \; year's \; Doomsday$$

This formula is particularly suitable for mental calculation, because it does not involve divisions and the procedure, as a recursive one, is easy to remember. So alternatively, to determine the doomsday it is also possible to add the last two digits of the year (_y_) to the quotient of the division between _y_ and 4. In fact, also following this simpler formula, the Doomsday of the year 1966 is Monday:

$$\Big[\Big(66 + \left \lfloor{\frac{66}{4}}\right \rfloor \Big) \; mod \; 7 \Big] + 3 = 8$$

### Step 3 aka finding the day of the week of the desired day
After you know the Doomsday of the year in question, the calculation of the day of the week of the desired day is very simple:
- identify the Doomsday closest to the chosen day,
- subtract the date of the nearest doomsday from the date to be identified,
- add the value of the Doomsday of the year,
- and finally report the result obtained in base 7;

Ok, let's start from the...what!? How do you identify the Doomsday closest to the chosen day?! Ok, do you remember the table with the doomsday for each month I talked about before? The only thing you have to do is deal with some comparison between the month and some simple computation between abs difference. A possible implementation of the formula using Ruby is:

{{< highlight ruby >}}
### Step 3 - find the day of the week

nearest = 9999999999

case month
	when 1
		if ((year % 4 == 0) and (year % 100 != 0)) or (year % 400 == 0)
			nearest = 4
		else
			[3, 31].each {
				|val|
				if (day-val).abs < nearest
					nearest = val
				end
			}
		end
	when 2
		ddays = [1, 29]
		if ((year % 4 == 0) and (year % 100 != 0)) or (year % 400 == 0)
			ddays = [7, 14, 21, 28]
		end
		ddays.each {
			|val|
			if (day-val).abs < nearest
				nearest = val
			end
		}
	when 3
		[7, 14, 21, 28].each {
			|val|
			if (day-val).abs < nearest
				nearest = val
			end
		}
	when 4
		nearest = 4
	when 5
		nearest = 9
	when 6
		nearest = 6
	when 7
		nearest = 11
	when 8
		nearest = 8
	when 9
		nearest = 5
	when 10
		nearest = 10
	when 11
		nearest = 7
	when 12
		nearest = 12
	else
		nearest = 9999999999
end
{{< / highlight >}}

And that's all: you are are done! In fact, the following three points of this third step can be computed with the following line:

{{< highlight ruby >}}
weekday = (day - nearest + doomsday) % 7
{{< / highlight >}}

And of course you can pretty print the result with something really dirty and compact like:

{{< highlight ruby >}}
puts "~~~~~~~~~~~~~~~~~~~~~~~"
puts ['January', 'February', 'March',
	  'April', 'May', 'June', 'July',
	  'August', 'September', 'October',
	  'November', 'December'][month-1]+
	  " "+day.to_s+", "+
	  year+" was "+['Monday', 'Tuesday',
	  				'Wednesday', 'Thusday',
	  				'Friday', 'Saturday',
	  				'Sunday'][weekday-1]
{{< / highlight >}}

For example, to calculate the September 13, 2011:

- the nearest Doomsday is 5/9;
- 13 - 5 = 8
- 8 + 1 = 9 (the 2011 Doomsday was Monday, so the corresponding value is 1)
- 9 mod 7 = 2 = Tuesday

Thank you everybody for reading!

[^wiki]: [John Horton Conway](https://en.wikipedia.org/wiki/John_Horton_Conway)
[^joke]: Do not worry: it's not an innate ability!! To improve his speed, he practices his calendrical calculations on his computer, which is programmed to quiz him with random dates every time he logs on.