

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


![Minimum example](/images/MinimalExampleLogTab.png)



#### Log page deluxe example:

In the tab "Log", you can see who changed or watched the current account. You can see, which date and time the changes were made. Further, you can see the old and also the new value of the changed attribute.

To see new and old passwords, you have to click on the text. Names, emails or descriptions are displayed directly.

If someone just watched the account or copied an attribute, then there will be no old and new value, like kmüller.


![Deluxe example](/images/DeluxeExampleLogTab.png)


#### Log page Epic example:

It would be useful, if we could filter the logs with the changed attribute. We would make this with a drop down with the different attributes which could change. More, we have here a column with the changed attribute and also the old and the new value, the date and time and the user.


![Epic example 1.1](/images/EpicExample1LogPage.png)


![Epic example 1.2](/images/EpicExample2LogPage.png)


![Epic example 1.3](/images/EpicExample3LogPage.png)



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


## What we need to discuss


If the user wants to see the password of an account in the account view, then he must click the password field. This is happening only in the frontend (card-show.hbs Line 43). If the user for example opens a specific account, we can detect this in the backend. But currently, we can't detect if he clicks on "Shows password". Because of that, we need to find a solution.

Maybe, we can add a new http call to the backend, so that we can write in the access log, if the user just watched the account or also the password.
