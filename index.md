---
layout: default
title: Meine GitHub Repositories
---

## GitHub Repositories

<ul id="repo-list"></ul>

<script>
fetch('https://api.github.com/users/volkansah/repos?type=owner&sort=updated&per_page=100')
  .then(response => response.json())
  .then(data => {
    let repoList = document.getElementById('repo-list');
    data.filter(repo => !repo.fork).forEach(repo => {
      let listItem = document.createElement('li');
      listItem.innerHTML = `<a href="${repo.html_url}">${repo.name}</a> - ${repo.description || 'Keine Beschreibung verfügbar'}`;
      repoList.appendChild(listItem);
    });
  })
  .catch(error => {
    console.error('Error:', error);
    let repoList = document.getElementById('repo-list');
    repoList.innerHTML = '<li>Fehler beim Laden der Repositories.</li>';
  });
</script>
