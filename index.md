---
layout: default
title: Meine GitHub Repositories
---

## GitHub Repositories

<ul id="repo-list"></ul>

<script>
document.addEventListener("DOMContentLoaded", function() {
  fetch('https://api.github.com/users/volkansah/repos')
    .then(response => response.json())
    .then(data => {
      console.log(data); // Überprüft, welche Daten von der API zurückgegeben werden
      let repoList = document.getElementById('repo-list');
      data.forEach(repo => {
        // Überprüft, ob das Repository ein Fork ist anhand des Namens
        if (repo.name.endsWith('-fork')) {
          // Optional: Forks hervorheben oder anders darstellen
          let listItem = document.createElement('li');
          let parentRepo = repo.parent ? repo.parent.full_name : 'Unbekanntes Repository';
          listItem.innerHTML = `<a href="${repo.html_url}">${repo.name}</a> - Geforkt von ${parentRepo}`;
          repoList.appendChild(listItem);
        } else {
          // Normale Anzeige für eigene Repositories
          let listItem = document.createElement('li');
          listItem.innerHTML = `<a href="${repo.html_url}">${repo.name}</a> - ${repo.description}`;
          repoList.appendChild(listItem);
        }
      });
    })
    .catch(error => {
      console.error('Error fetching repos:', error);
      document.getElementById('repo-list').innerHTML = '<li>Fehler beim Laden der Repositories.</li>';
    });
});
</script>
