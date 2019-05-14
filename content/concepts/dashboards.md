---
title: Dashboards
aliases: ["/ref/dashboards"]
---

Humio can display a collection of different widgets, each with their
own set of queries, in a browser typically used on a passive monitor
without requiring authenticating as a user. The URL in itself is then
accepted as authentication and when accessed in that manner the
dashboard is not interactive: You can only execute the exact queries
that are saved on the shared dashboard.

This is great for "big screens" for display of statistics on your
system for use in the "war room" or perhaps for public statistics in
the lobby.

### Creating a new dashboard

A dashboard can be made up of anything you can make up on the search
page, but usually consists of a number of timecharts and tables all
based on "live queries". If you add a non-live query to a dashboard,
the widget will not get updated.

To create a new dashboard, click "Save as" on on the search page of
the first widget you want to have in the dashboard.

To add more widgets to an existing dashboard, click "Save as" on on
the search page of any search you want to add to an existing
dashboard. The dialog that shows there allows you to select which
dashboard to add the widget to.

### Editing a dashboard

You can resize and position the widgets by dragging them at the title
or borders.

You can edit the contents of a widget, including the style of the
chart etc. by clicking the title of the widget, then editing the
search just like any other search, and then pressing "Save" to save
the modified widget, selecting to replace the old one in the dashboard
if that is what you want.

### Sharing a dashboard in read-only mode

A dashboard can be shared by clicking the "share" button in the "..."
menu and creating a new share link. Share links have names to help you
remember where they are being put to use.

The share links always shows the latest version of the dashboard. If
you share the dashboard, then later edit the widgets in the dashboard,
the read-only share links being shown on other screens will show the
latest version of the dashboard.

### Exporting, Copying, Importing

When creating a new dashboard you can start from a single query or you
can clone an existing dashboard, also from other repositories in the
same Humio cluster, or import a template. You create the template by
exporting an existing dashboard to a file. Such templates can be
imported into both the same Humio cluster as well as into another
Humio cluster.
