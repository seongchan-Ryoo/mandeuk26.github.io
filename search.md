---
layout: page
title: Search
---

<!-- Html Elements for Search -->
<div id="search-container">
	<input type="text" id="search-input" placeholder="Type here ...">
	<hr/>
	<div id="results-container"></div>
</div>

<!-- Script pointing to jekyll-search.js -->
<script src="{{ site.baseurl }}public/js/jekyll-search.js" type="text/javascript"></script>

<script type="text/javascript">
	SimpleJekyllSearch({
		searchInput: document.getElementById('search-input'),
		resultsContainer: document.getElementById('results-container'),
		json: '{{ site.baseurl }}search.json',
		searchResultTemplate: '<div><a href="{url}">{title}</a></div>',
		noResultsText: 'No results found',
		limit: 10,
		fuzzy: true,
	})
</script>
