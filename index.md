---
layout: default
title: Meine GitHub Repositories
---

## GitHub Repositories

<ul id="repo-list"></ul>

<script>
fetch('https://api.github.com/users/volkansah/repos')
  .then(response => response.json())
  .then(data => {
    let repoList = document.getElementById('repo-list');
    data.forEach(repo => {
      // Überprüft, ob das Repository ein Fork ist anhand des Namens
      if (repo.name.endsWith('-fork')) {
        // Optional: Forks hervorheben oder anders darstellen
        let listItem = document.createElement('li');
        listItem.innerHTML = `<a href="${repo.html_url}">${repo.name}</a> - Geforkt von ${repo.parent.full_name}`;
        repoList.appendChild(listItem);
      } else {
        // Normale Anzeige für eigene Repositories
        let listItem = document.createElement('li');
        listItem.innerHTML = `<a href="${repo.html_url}">${repo.name}</a> - ${repo.description}`;
        repoList.appendChild(listItem);
      }
    });
  });


</script>
