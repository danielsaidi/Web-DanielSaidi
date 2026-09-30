---
title: onSelectRow fix for jqGrid
date:  2010-10-07 12:00:00 +0100
tags:  web
icon:  javascript
---

I love the [jqGrid jQuery plugin](http://www.trirand.com/blog/). If you haven't tried it, you should. However, `onSelectRow` doesn't work that well. Let's fix it.

When you look at online demos that describe how to edit a grid row, you may have noticed that after editing a row, you must select a new one before you can edit the first one again.

To fix this, I replaced the original `onSelectRow` implementation:

```javascript
onSelectRow: function(id) {
   if (id && id!==lastsel) {
      jQuery('#rowed3').jqGrid('restoreRow',lastsel)
      jQuery('#rowed3').jqGrid('editRow',id,true);
      lastsel=id;
   }
}
```

...with this one:

```javascript
onSelectRow: function(id) {
   if (id) {
      if (id !== lastsel) {
         articleGrid.jqGrid('restoreRow', lastsel);
         articleGrid.jqGrid('editRow', id, true);
         lastsel = id;
      } else {
         articleGrid.jqGrid('restoreRow', lastsel);
         lastsel = "";
      }
   }
}
```

With this tiny adjustment, the grid now behaves a lot more like I want it to.