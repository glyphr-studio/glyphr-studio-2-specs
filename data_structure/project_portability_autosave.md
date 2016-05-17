# Project portability / Project auto-save
## Spec Status: Crazy idea, we're probably not doing all of this

HTML5 Local Storage can be used to auto-save Glyphr Studio projects, or project edit history.  Additionlly, a downloadable project file  will enable projects to be shared or opened on different computers.  Glyphr Studio should recognize a single project, even across edits on different computers.

### Example
In this scenario I'll be using pseudo code notation / simplified examples.  Hierarchically, edits to a single project can be categorized as:
```
computer.browser.localstorage.glyphrstudio.project.edits = []
```
For simplicity I'll shorten this to 
```
computer.ls.gs.project.edits
```
In this example I'll also assume Glyphr Studio stores a maximum of **5 edits per project per computer**... this is arbitrary.  Actual implementation could be optimized a number of different ways to increase this.

Edit0, Edit1, Edit2... each 'Edit' represents some state of the project.  Think stringified JSON data... but again, that's an implementation detail.  In our example, though, the Glyphr Studio Project File is equivalent to the most recent Edit entry.

### Quick Scenario
Joe uses Glyphr Stuido to design many different fonts, each one contained within a Glyphr Studio Project.  Joe has two computers, which we'll call 'Work' and 'Home'.  One of Joe's projects is called 'Proj123' - but he has many others, for example, 'Proj456' and 'Proj789'.

Joe has a dropbox where he stores the most recent versions of his project files.  Most days, he loads a project and makes a few edits at home, saving the project file back to dropbox when he's done.  Then he goes to work, and over lunch, he loads the same project from dropbox, makes a few more edits, then saves a save file back to dropbox.

### Data model
Using our simplified data notation, let's walk through Joe's day.

**Joe starts a new project 'proj123' and makes 3 edits at home**
```
home.ls.gs = {
	'proj123': [Edit2, Edit1, Edit0],
    'proj456': [...],
    'proj789': [...]
}
```

**Joe loads proj123 at work, and makes 3 more edits**
```
work.ls.gs = {
	'proj123': [Edit5, Edit4, Edit3, Edit2],
    'proj111': [...]
}
```

**At home again, Joe loads proj123**
```
home.ls.gs = {
	'proj123': [Edit5, Edit2, Edit1, Edit0],
    'proj456': [...],
    'proj789': [...]
}
```

**And makes one edit**
```
home.ls.gs = {
	'proj123': [Edit6, Edit5, Edit2, Edit1, Edit0],
    'proj456': [...],
    'proj789': [...]
}
```

**Making one more edit will put us over our arbitrary 5 max**
```
home.ls.gs = {
	'proj123': [Edit7, Edit6, Edit5, Edit2, Edit1],
    'proj456': [...],
    'proj789': [...]
}
```

**The next day Joe goes straight to work, but loads proj123 and makes 2 edits**
```
work.ls.gs = {
	'proj123': [Edit9, Edit8, Edit7, Edit5, Edit4],
    'proj111': [...]
}
```

### Discussion
* If each project has a unique ID that is both saved with the save file, and used to identify local storage edit history, projects can be identified across computers across edits.
* We should consider exposing in Glyphr Studio some sort of "Clear All Project History" or show a list of all projects locally stored to clear individual project histories.
* HTML5 has roughly 5mb shared storage.  A v1 sample project with ~400 glyphs has a JSON text file of about 3kb. Save file optimization / size will determine our edit history target.
	* v1 saved whole project JSON files per undo-step.  Implementing this as diff would be much more efficient.
* It could be possible to save edit history with the project file and load it on whatever computer.
* We should also consider some sort of "Fork this project" action, that basically assigns a new unique ID to the project, branching it from it's previous edit history.
