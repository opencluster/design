# Console

The console is a web-based service that provides visual functionality to administrators of the system.

## Custom front-end

By default when the system is first installed, it provides a standard template.  This shows some metrics and monitoring, along with some standard operational functionality.  However this can be modified to suit the requirements of the organisation.

It can also be used to handle incidents in an interactive way, but this generally has to be planned for.

> An example might be that if a problem is detected and a human needs to decide what action to take, it can present that question to the staff (by activating a part of the page that is normally not visible).

## Page modifications

The page layout is available to be edited, and is in a format that allows for very flexible customisation.  There will be a large number of templates available.

Each section in the main page can include:

* Notification banners
* Static panels
* Hidden panels that are displayed when something is triggered
* User customisation also (each user can minimise or remove panels they do not like.
* external pages (referenced like iframes)
* Links to other services.


Panel content can reference LUA scripts that can derive information, or can be in a format that retrieves information either from an app, or from internal config data reference.


### Page updating and control options.

One annoying aspect could be that as an incident is occuring, multiple issues are occuring, and so the page layout may be changing constantly.  There would therfore be some control over that.

Three buttons would exist on the page (normally in the top corner).

 * One button sets it in a mode where it is constantly updating.
 * The second mode is that is in a more interactive mode (which is the standard setting).  When the user is moving the mouse or typing, it does not change the layout of the page.  However, the page (likely the background) will flash slightly to indicate that there is changes it wants to present.
 * The third mode is the pause mode, which locks it into the current state.  The button itself will flash if there is changes that want to be presented.

