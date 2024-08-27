---
layout: default
title: Meine GitHub Repositories
---

## GitHub Repositories

<ul id="repo-list"></ul>

<script>
fetch('https://api.github.com/users/volkansah/repos?type=owner')
  .then(response => response.json())
  .then(data => {
    let repoList = document.getElementById('repo-list');
    data.forEach(repo => {
      let listItem = document.createElement('li');
      listItem.innerHTML = `<a href="${repo.html_url}">${repo.name}</a> - ${repo.description}`;
      repoList.appendChild(listItem);
    });
  });

</script>
