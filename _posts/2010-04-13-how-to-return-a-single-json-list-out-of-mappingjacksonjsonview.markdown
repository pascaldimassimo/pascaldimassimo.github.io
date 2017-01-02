---
layout: post
title: 'How to return a single JSON list out of MappingJacksonJsonView'
---

I really like to build REST web services with Spring Web MVC. The ContentNegotiatingViewResolver allows to easily render an object or a collection of objects to many representations, like XML, JSON or good old HTML.

In a recent project, I was using the MappingJacksonJsonView to render my objects to JSON. It works pretty well. Just put your POJO objects in a ModelAndView and it will be transformed to a JSON map. It also works with collections of objects. If you add an ArrayList of objects to a ModelAndView, MappingJacksonJsonView will transform it to a JSON list containing a map for each object. However, MappingJacksonJsonView always wraps the objects you put in the ModelAndView inside a JSON map. It makes sense in the majority of cases, but when you want to return a single list, it will be embedded inside a map, which was not what I want.

So, by default, this code:

```java
ModelAndView mav = new ModelAndView();
mav.addObject(listOfObjects);
return mav;
```

will produce this JSON representation:

```json
{"objectList" : [{"name":"object1"}, {"name":"object2"}]}
```

But what I would like to have is simply:

```json
[{"name":"object1"}, {"name":"object2"}]
```

In order to do that, I had to subclass the MappingJacksonJsonView class to override the filterModel object. This method returns the map of all objects to be transformed to JSON. But since it always returns a map, the final output is always a JSON map. So what I did in my subclass is, after executing the parent's filterModel method, check if the map contains a single object and if that is the case, return that single object out of the map.

```java
import java.util.Map;

import org.springframework.web.servlet.view.json.MappingJacksonJsonView;

/**
 * This class will make sure that if there is a single object to
 * transform to JSON, it won't be rendered inside a map.
 */
public class MappingJacksonJsonViewEx extends MappingJacksonJsonView
{
   @SuppressWarnings("unchecked")
   @Override
   protected Object filterModel(Map<String, Object> model)
   {
      Object result = super.filterModel(model);
      if (!(result instanceof Map))
      {
         return result;
      }

      Map map = (Map) result;
      if (map.size() == 1)
      {
         return map.values().toArray()[0];
      }
      return map;
   }
}
```

Voilà! Now it only returns the list.
