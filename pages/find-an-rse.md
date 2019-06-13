---
title: Find a Research Software Engineer
sitemap: false
permalink: /find-an-rse/
not_editable: true
---

<form action="{{ site.baseurl }}/find-an-rse" method="get">
	<input type="search" name="q" id="search-input" placeholder="Search {{ site.title }}?" style="margin-top:5px" autofocus>
	<input type="submit" value="Search" style="display: none;">
</form>

<p><span id="search-process">Loading</span> results <span id="search-query-container" style="display: none;">for "<strong id="search-query"></strong>"</span></p>
<ul id="search-results"></ul>

<script src="{{ site.baseurl }}/assets/js/lunr.min.js"></script>
<script src="{{ site.baseurl }}/assets/vendor/jquery/jquery.min.js" ></script>
<script>
(function() {
  window.data = {}
  $.getJSON("{{ site.baseurl }}/data.json", {
    format: "json"
  })
   .fail(function(error) {
      console.log("Error", error);
   })
   .done(function(data) {
      $.each(data, function(key, value ) {
        window.data[key] = value;
      });
   });
  function loadSearch() {
    console.log(window.data)
    var fileref = document.createElement('script')
    fileref.setAttribute("type","text/javascript")
    fileref.setAttribute("src", "{{ site.baseurl }}/assets/js/search.js")
    document.getElementsByTagName("head")[0].appendChild(fileref)
  }
  setTimeout(loadSearch, 1000);
})();


</script>
