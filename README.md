

# Access and changelog conception


### Current situation

Currently, you can save your passwords in cryptopus. You can add new accounts with name, username, password and description. You can also share your team with multiple passwords and folders with other members. Then everybody who's added to this team can CRUD the values.


### Motivation

If multiple members work with one team in cryptopus, it can quickly become confusing. For example, someone updates a password for an account, but someone has to use the old password. Because of this, it would be useful to have an access- and changelog, where you can see, who updated what with which value.



### How you can access the page

There will be a new tab with the name "Log", when you open the detail view of an account. The account page would then look something like this:


![Account page with new tab](/images/EncryptablesPageWithNewTab.png)



### Versions

#### Log page minimal example:

In the new tab "Log", you can see who accessed the account. This would be, if someone opens the account page, copied an attribute or edited something. Additionally, you can see, which date and time the access has been.


![Minimum example](/images/MinimalLogPage1.png)

![Minimum example hover](/images/MinimalLogPage2.png)


#### Log page deluxe example:

In the tab "Log", you can see who changed or watched the current account. You can see, which date and time the changes were made. Further, you can see the old and also the new value of the changed attribute.

To see new and old passwords, you have to click on the text. Names, emails or descriptions are displayed directly.

If someone just watched the account or copied an attribute, then there will be no old and new value, like kmüller.


![Deluxe example](/images/DeluxeLogPage.png)


#### Log page Epic example:

It would be useful, if we could filter the logs with the changed attribute. We would make this with a drop down with the different attributes which could change. More, we have here a column with the changed attribute and also the old and the new value, the date and time and the user.


![Epic example 1.1](/images/EpicExample1LogPage.png)


![Epic example 1.2](/images/EpicExample2LogPage.png)


![Epic example 1.3](/images/EpicExample3LogPage.png)


### Following mockups are not finished and have been paused

#### Log page ultra example:

This is one of the most detailed version of the feature, which we could make. The user can define somewhere in the settings (Profile), his default filter. This filter would be active, if the user opens a new log page. In this version, he could also search for specific users or a date. Of course, the user can change the filter with the dropdown menu.


![Ultra example default filter](/images/UltraExample1LogPage.png)


![Ultra example open dropdown menu](/images/UltraExample2LogPage.png)


![Ultra example no filter](/images/UltraExample3LogPage.png)


![Ultra example filter and name search](/images/UltraExample4LogPage.png)



### My activity page

Would be useful, to see all your recent changes. These changes, you can see in the profile. There would be a new tab, called "Changes"


![Profile info page](/images/ProfileInfo.png)


#### My activity minimal example:


Simple, you can see the date, time and place from your recent changes or accesses. The path from the place is clickable, so you can easily move directly to the specific account.


![Minmal changes page 1](/images/MinimalExampleProfileChanges.png)


#### My activity deluxe example:


In the second example, you can also see date, time and place from your changes or accesses. Here, you can click the place-link and move directly to the specific account too. More, you can see your changed or watched attribute and the old and new value of an attribute. To see changed passwords values, you need to click on the text "Show PW".


![Deluxe changes page](/images/DeluxeExampleProfileChanges.png)



## New dashboard

Some people use mostly the same passwords, so it's an effort to search them in all the other passwords. Therefore, it would be useful, to have the recent used passwords directly on the dashboard. Design example:


#### Dashboard example 1

We have a new column with maximal 5 recent passwords. These are direct links to the passwords and not to the teams. All the other teams are below.


![Dashboard example 1](/images/DashboardExample1.png)


#### Dashboard example 2

This dashboard looks similar to the current dashboard. But here, we would not list the teams by name (A to Z), but with date of use. The recent used team would be at position 1. Here, the link would be to the team and not to a password.


![Dashboard example 2](/images/DashboardExample2.png)


## Technology


Useful tool for changelog and resetting passwords/username to old values:


[Paper Trail](https://github.com/paper-trail-gem/paper_trail)


With this, it is easy to get older values of an attribute and so we can easily generate a changelog table.

We can get all the data in backend and refresh the frontend with the newest data.


This is the basic how it works

```

widget = Widget.find 153

widget.name # 'Doobly'


# Add has_paper_trail to Widget model.


widget.versions # []

widget.update name: 'Wotsit'

widget.versions.last.reify.name # 'Doobly'

widget.versions.last.event # 'update'

widget.versions.length # 1

```


##### Rough process

- Add PaperTrail to Gemfile: ```gem 'paper_trail'```

- Add a versions table to the database ``` bundle exec rails generate paper_trail:install [--with-changes] ```

- Migrate the database ``` rails db:migrate ``` or ```bundle exec rake db:migrate```

- Add ```has_paper_trail``` to the model to track

Then basic usage:

Our models now have a versions method which returns the "paper trail" of changes to our model.

```

widget = Widget.find 42

widget.versions

# [<PaperTrail::Version>, <PaperTrail::Version>, ...]

```

Once we have a version, we can find out what happened:

```

v = widget.versions.last

v.event # 'update', 'create', 'destroy'. See also: Custom Event Names

widget.versions.reify.attributeName # Old value from the attribute

```

Get older versions than the second latest

```

widget = Widget.find 42

version = widget.versions[-2] # assuming widget has several versions

previous_version = version.previous

next_version = version.next

```

With ``` version = widget.versions[-2] ``` we get the second last version. If we count up the number, we can get every version. Useful would be a for loop with the length from ``` versions.length ```. 

Paper trail has a function to register, who did an update of a value. This is done with the versions.whodunnit, but in my test, this variable was always nil. Maybe we need to find first a solution for this. 

### New request
To get the data for the log, we need a new request. This request will be called, when the user opens the new tab "Log". The new request url would be ``` /api/encryptables/id/logs ```. In frontend/router.js, we would define a new route for the encryptables. This route would call the specific method in the [log_controller](#new-controller).

### New controller
We need a new controller, we would name it "log_controller". This controller is called, when the request [/api/encryptables/id/log](#new-request) is called. The log_controller prepares all the data for the log (user, date&time,...). How we can get these data is described above, we use [Papertrail](#technology). Then, we would return the data, maybe in json format. The returning is similar to the returning from an encryptable (see encryptables_controller.rb). 

#### Information for deluxe, epic and ultra editions: 
In this 3 versions, we have the old and the value from the password. We don't want to display the passwords directly. For this "problem", we can use a similar way as we use it in the encryptables view. There, we display the password only if the user clicks on the text "show password". But we send the password together with the username from the backend to the frontend. For the logs, we would send all attributes to the frontend too and there, we would then hide the 2 values from the password.

### Pagination
When you have shared a password with multiple persons, then the log page can quickly become full of entries. If there are for example 200 entries, then the page needs a time to be loaded. To reduce the loading time, we should use pagination. This means, that we load maybe the first 15 logs and if the user scrolls down, we load the next 15 entries. With this method, we can save more time. 
We can use a tool, called [will_paginate](https://github.com/mislav/will_paginate). This takes us the work for this problem.
Another option would be to split the logs into groups, then we would load the first group, then when the user scrolls, we load the next part. We could change this in the url. [Example for this option](https://stevepolito.design/blog/rails-infinite-scrolling-blog-roll/). But the first method is better for us, because we want all logs in one table and not in multiple. 
