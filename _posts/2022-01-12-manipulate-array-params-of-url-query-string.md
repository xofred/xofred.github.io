---
title:  "Manipulate Array Params of URL Query String"
tags: javascript

---

*Tldr: `URL.searchParams`*

<video width="320" height="240" controls>
  <source src="https://user-images.githubusercontent.com/2174219/149091462-5456ea2f-ff96-4490-a171-97d248205fee.mp4" type="video/mp4">
</video>

```erb
<!-- some legacy erb -->

<dl>
  <% if @selected_price_ranges %>
    <% @selected_price_ranges.each do |content| %>
      <dd>
        <%= content %>
        <span class="icon-close"
              data-key="price_ranges[]"
              data-value="<%= content %>"></span>
      </dd>
    <% end %>
  <% end %>
</dl>

<script>
$(function () {
  $('dl dd span.icon-close').on('click', function(e){
    if (typeof URL !== 'undefined') {
      let url = new URL(window.location.href)
      let key = $(this).data('key')
      //console.log(url.searchParams.getAll(key))

      let newValues = url.searchParams.getAll(key)
                         .filter( value => value !== $(this).data('value') )
      url.searchParams.delete(key)
      for (newValue of newValues) {
        url.searchParams.append(key, newValue)
      }
      //console.log(url.searchParams.getAll(key))
      window.location = url
    } else {
      console.log(`Your browser ${navigator.appVersion} does not support URL`)
    }
  });
})
</script>
```

### Reference

[Easy URL Manipulation with URLSearchParams](https://developers.google.com/web/updates/2016/01/urlsearchparams)

[How can I delete a query string parameter in JavaScript?](https://stackoverflow.com/questions/1634748/how-can-i-delete-a-query-string-parameter-in-javascript)

[How to add local video file in markDown md file](https://stackoverflow.com/questions/46273751/how-to-add-local-video-file-in-markdown-md-file#46296699)
