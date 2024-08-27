---
layout: default
title: Meine GitHub Repositories
---

## GitHub Repositories

<div class="container">
    <ul id="repo-list" class="list-unstyled row"></ul>
    <div id="pagination" class="text-center mt-4">
        <button id="prev" class="btn btn-primary" disabled>Vorherige</button>
        <span id="page-info" class="mx-2"></span>
        <button id="next" class="btn btn-primary">Nächste</button>
    </div>
</div>

<script>
let currentPage = 1;
const perPage = 12; // Anzeigen von 12 Repositories pro Seite
let totalRepos = 0;

function fetchRepos(page) {
  fetch(`https://api.github.com/users/volkansah/repos?type=owner&sort=updated&per_page=${perPage}&page=${page}`)
    .then(response => {
      totalRepos = parseInt(response.headers.get('X-Total-Count') || '0');
      return response.json();
    })
    .then(data => {
      let repoList = document.getElementById('repo-list');
      repoList.innerHTML = '';

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
              <button class="btn btn-primary" data-toggle="modal" data-target="#repoModal-${index}">Details anzeigen</button>
            </div>
          </div>

          <div class="modal fade" id="repoModal-${index}" tabindex="-1" role="dialog" aria-labelledby="repoModalLabel-${index}" aria-hidden="true">
            <div class="modal-dialog modal-lg" role="document">
              <div class="modal-content">
                <div class="modal-header">
                  <h5 class="modal-title" id="repoModalLabel-${index}">${repo.name}</h5>
                  <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                    <span aria-hidden="true">&times;</span>
                  </button>
                </div>
                <div class="modal-body">
                  <p><strong>Beschreibung:</strong> ${repo.description || 'Keine Beschreibung verfügbar'}</p>
                  <p><strong>Link zum Repository:</strong> <a href="${repo.html_url}" target="_blank">${repo.html_url}</a></p>
                </div>
                <div class="modal-footer">
                  <button type="button" class="btn btn-secondary" data-dismiss="modal">Schließen</button>
                </div>
              </div>
            </div>
          </div>
        `;
        repoList.appendChild(listItem);
      });

      updatePagination(filteredData.length);
    })
    .catch(error => {
      console.error('Error:', error);
      let repoList = document.getElementById('repo-list');
      repoList.innerHTML = '<li>Fehler beim Laden der Repositories.</li>';
    });
}

function updatePagination(filteredCount) {
  const totalPages = Math.ceil(totalRepos / perPage);
  document.getElementById('page-info').textContent = `Seite ${currentPage} von ${totalPages}`;
  document.getElementById('prev').disabled = currentPage === 1;
  document.getElementById('next').disabled = currentPage === totalPages || filteredCount <= perPage;
}

document.getElementById('prev').addEventListener('click', () => {
  if (currentPage > 1) {
    currentPage--;
    fetchRepos(currentPage);
  }
});

document.getElementById('next').addEventListener('click', () => {
  const totalPages = Math.ceil(totalRepos / perPage);
  if (currentPage < totalPages) {
    currentPage++;
    fetchRepos(currentPage);
  }
});

fetchRepos(currentPage);
</script>
