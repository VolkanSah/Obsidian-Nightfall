---
layout: default
title: Meine GitHub Repositories
---

## GitHub Repositories

<div class="container">
    <ul id="repo-list" class="list-unstyled row"></ul>
    <div id="load-more-container" class="text-center mt-4">
        <button id="load-more" class="btn btn-primary">Mehr laden</button>
    </div>
</div>

<script>
let currentPage = 1;
const perPage = 12; // Laden von 12 Repositories pro Klick
let totalRepos = 0;
let loadedRepos = 0;

function fetchRepos(page) {
  fetch(`https://api.github.com/users/volkansah/repos?type=owner&sort=updated&per_page=${perPage}&page=${page}`)
    .then(response => {
      totalRepos = parseInt(response.headers.get('X-Total-Count') || '0');
      return response.json();
    })
    .then(data => {
      let repoList = document.getElementById('repo-list');

      let filteredData = data.filter(repo => {
        return !repo.fork && 
               repo.name !== 'volkansah.github.io' && 
               repo.name !== 'VolkanSah';
      });

      filteredData.forEach((repo, index) => {
        let listItem = document.createElement('li');
        listItem.className = 'col-md-4';

        listItem.innerHTML = `
          <div class="card mb-4">
            <div class="card-body">
              <h5 class="card-title">${repo.name}</h5>
              <p class="card-text">${repo.description || 'Keine Beschreibung verfügbar'}</p>
              <button class="btn btn-primary" data-toggle="modal" data-target="#repoModal-${loadedRepos}" onclick="loadReadme('${repo.full_name}', ${loadedRepos})">Details anzeigen</button>
            </div>
          </div>

          <div class="modal fade" id="repoModal-${loadedRepos}" tabindex="-1" role="dialog" aria-labelledby="repoModalLabel-${loadedRepos}" aria-hidden="true">
            <div class="modal-dialog modal-lg" role="document">
              <div class="modal-content">
                <div class="modal-header">
                  <h5 class="modal-title" id="repoModalLabel-${loadedRepos}">${repo.name}</h5>
                  <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                    <span aria-hidden="true">&times;</span>
                  </button>
                </div>
                <div class="modal-body" id="repoContent-${loadedRepos}">
                  <p>Lade README...</p>
                </div>
                <div class="modal-footer">
                  <a href="${repo.html_url}" target="_blank" class="btn btn-primary">Zum Repository</a>
                  <button type="button" class="btn btn-secondary" data-dismiss="modal">Schließen</button>
                </div>
              </div>
            </div>
          </div>
        `;
        repoList.appendChild(listItem);
        loadedRepos++;
      });

      if (loadedRepos >= totalRepos) {
        document.getElementById('load-more').style.display = 'none';
      }
    })
    .catch(error => {
      console.error('Error:', error);
      let repoList = document.getElementById('repo-list');
      repoList.innerHTML = '<li>Fehler beim Laden der Repositories.</li>';
    });
}

function loadReadme(repoFullName, index) {
  fetch(`https://api.github.com/repos/${repoFullName}/readme`, {
    headers: { 'Accept': 'application/vnd.github.v3.html' }
  })
  .then(response => response.text())
  .then(data => {
    document.getElementById(`repoContent-${index}`).innerHTML = data;
  })
  .catch(error => {
    document.getElementById(`repoContent-${index}`).innerHTML = '<p>README konnte nicht geladen werden.</p>';
  });
}

document.getElementById('load-more').addEventListener('click', () => {
  currentPage++;
  fetchRepos(currentPage);
});

// Initiales Laden der ersten Seite
fetchRepos(currentPage);
</script>
