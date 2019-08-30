---
layout: post
title: "How to plan your daily activity with Python"
categories: [coding, algorithms]
tags: [python, algorithms, scheduling, task]
---

### The problems of life
Each of us has dreams, aspirations, hobbies, interests, but also hundreds of deadlines, a thousand commitments, ten thousand thoughts, a hundred thousand different problems to cope with every day. I am a computer scientist and personally all these things in my life translate into a huge pile of [things/books/articles/guide/blogs] to read, which by the way are very often interrelated. The situation is more or less this:

![books](https://i.imgur.com/ojx9vzi.jpg)

The problem is that often in the presence of many things to do we lose more time deciding how to do them and when to decide to start them. I formalized a simple method to handle all my stuff.

### How to do everything
The short answer is: you can not. The innocent answer is: you can, you just have a good plan that takes into account every moment of time you can cut out in your day already full of commitments. The sincere answer is: yes, but it involves an incredible amount of time to _schedule_ everything. In times of orchestrators and dev (or no? n.d.r.) ops, I have finally decided to brush up on a logic that I realized during the years of university studies to prepare the exams: of course, since I'm a developer, I have implemented a version - preferring for this time Python instead of Go (with regret, I swear) because I had already some snippets ready.

### What do you need?
Patience and a couple of hours to implement your solution. Then:

- Python (you do not even need a virtualenv, if not for two additional boundary libraries - not necessary for the strict logic of implementation of the program);
- A list of _TODOs_ (for instance, 'Read Computer Networks', 'Workout', 'Write a new article', etc);
- A unit of measure for each of the activities on the list that is _small enough_ (what I mean? for example, for reading material it could be pages to read, for a generic activity a number that represent a percentage to complete the task);

### A real case
I recently bought a TCP/IP book to study and learn a little bit more about computer networks. The problem is that I bought 3 more books together with this one (damn it, Amazon!). I would like to end my reading in a reasonable number of days. However, I have other commitments that take away precious time: unfortunately, I can not spend all my time reading this book. The book is a simple object to formalize: just write on a file some aspects to calibrate the way to divide the pages to be addressed daily. Let's formalize a book.

### Example of book
{% highlight json %}
  'Read TCP/IP': {
    'totalPage'  : 920,
    'pageForDay' : 7,
    'pageForMinute': 0.20,
    'totalCompleted' : 89,
    'totalRemaining' : 920,
    'percCompleted'  : 9.6,
  }
{% endhighlight %}

| Option         | Description |
| -------------- | ---------------------------------------------------------------------------------------- |
| totalPage      | Total page of the book / Total _amount_of_time_ you need to complete the activity        |
| pageForDay     | Number of page you want to read each day (you have to be honest with you, do not overdo) |
| pageForMinute  | A text could be more difficult then others: how many page can you complete in a minute?  |
| totalCompleted | The number of page you have completed (this value change over time)                      |
| totalRemaining | The remaining number of page (redudant, used only for clarity)                           |
| percCompleted  | Percentage of completed activity, see lines above                                        |

You can easily imagine the other formalizations within a dictionary whose keys are the titles of my activities.

### Let's code!
First, remove work hours from your typical day: for instance, after 8 hours of work, mostly because I work outside of my city - I can spent only 1 hour and half for my personal interests. After that I do binge watching on Netflix / I go out with friends / generally I stop studying or working or do useful things XD
I also suggest you setup a fixed _maximum_ number of different activities (books, etc) to assimilate what you read / do in a correct way. I use 4 as my _maximum_.

{% highlight python %}

# setup start hour
startHour = 18
startDay  = datetime.datetime.today()
startDay.replace(hour=startHour, minute=00, second=00)

# setup working minutes
workingMinutes = 90

# setup max number of different activities
workingMaximumThemes = 4

{% endhighlight %}

After that, as long as each activity in the activity list contains activities that are not yet 100% complete, you only have to loop over your list of activities, pickup a random maximum number of activities, make some maths to update the number of page you have to read / more generally update the percentange of activity you can complete in the amount of minutes for the day, create the event and schedule it - I use GCal (I wrote a more complex solution [here](https://github.com/made2591/google-task-gtd), but you can use Trello (I wrote a lib available [here](https://github.com/made2591/trello2google), or Todoist, and so on.

{% highlight python %}

# randomize keys in activities dict
pickedUpActivities = activities.keys()
random.shuffle(pickedUpActivities)

# for a maximum amount of time in current day
for activityName in pickedUpActivities[:workingMaximumThemes]:

  # start day hour
  startWorkHour = startDay.replace(hour=startWorkHour, minute=00, second=00)
  endWorkHour   = startWorkHour + datetime.timedelta(minutes=int(workingMinutes/workingMaximumThemes))

  # update properties of activity object in dict
  activities[activityName]['totalCompleted'] += activities[activityName]['pageForDay']
  ...

  # look in https://github.com/made2591/google-task-gtd for more detail
  createGcalEvent(service, calendar_id, event_template)

  # update for next task
  startWorkHour = startDay.replace(hour=startWorkHour, minute=00, second=00)
  endWorkHour   = startWorkHour + datetime.timedelta(minutes=int(workingMinutes/workingMaximumThemes))

{% endhighlight %}

You've done! The only things to do is inserting the loop above in another loop over the entire activity list and break this external loop when all activities contained are 100% complete. You may have notice that the properties ```pageForMinute``` is not used. When you have a very long and complex tasks list, sooner or later, following the illustrated algorithm, the list of activities to be completed yet will contain fewer activities than the maximum you can manage in one day. It is also true that the number of pages / percentage of activity that you want to complete daily is related to the number of minutes available in your average day __and__ the maximum number of context switches (workingMaximumThemes) you are willing to do (if not, it should so fine tune your activity list). This is why I defined a __speed__ (```pageForMinute```) parameter for each activity.

### Adaptive effort
I generally use to __decrease__ the workingMaximumThemes setting it to the ```max(workingMaximumThemes, len(pickedUpActivities[:workingMaximumThemes]))```, __increase__ the number [pageForDay to read / percentage of activity to complete] multiply this by a factor provided by the __speed__ (```pageForMinute```) parameter of each activities.

{% highlight python %}

# randomize keys in activities dict
pickedUpActivities = activities.keys()
random.shuffle(pickedUpActivities)

# for a maximum amount of time in current day
for activityName in pickedUpActivities[:workingMaximumThemes]:

  # adaptive speed part
  activities[activityName]['pageForDay'] = Math.max(
    int(int(workingMinutes/workingMaximumThemes) * book['pageForMinute'])
    int(int(workingMinutes/workingMaximumThemes) * book['pageForMinute'])
  )

  # start day hour
  startWorkHour = startDay.replace(hour=startWorkHour, minute=00, second=00)
  endWorkHour   = startWorkHour + datetime.timedelta(minutes=int(workingMinutes/workingMaximumThemes))

  # update properties of activity object in dict
  activities[activityName]['totalCompleted'] += activities[activityName]['pageForDay']
  ...

  # look in https://github.com/made2591/google-task-gtd for more detail
  createGcalEvent(service, calendar_id, event_template)

  # update for next task
  startWorkHour = startDay.replace(hour=startWorkHour, minute=00, second=00)
  endWorkHour   = startWorkHour + datetime.timedelta(minutes=int(workingMinutes/workingMaximumThemes))

{% endhighlight %}

Let's take an example: I have 90 minutes, and 3 activities (assuming 3 books to complete). One of the books has double the page of the first, the third has the same number of page of the second plus 50. I can handle 3 activities each days, I want to read 10 page for each books. In a situation of maximum entropy (uniformity of difficulty in reading, etc.), if in 30 minutes I read 10 page for a book, in 90 I read a total of 30 pages, 10 for each books. After a months, I finish the first book. What happens? It happens that I can handle more then 20 pages in 90 minutes. I have to change the number of pages I can read for each books: the best way is to distribute the time in a uniform way and multiply 45 minutes for the __speed__ for each of the remaining books. You can of course custom the algorithm to handle different speed, different priorities, and so on: a draft of what I did manipulating ```datetime``` is [here](https://github.com/made2591/google-task-gtd) (really bad code, sorry for that)

### Plus
To easily ignore holidays and access advanced features on objects from the datetime library I found it very useful [workalendar](https://github.com/peopledoc/workalendar) and relativedelta (from dateutil)

Thank you for reading!