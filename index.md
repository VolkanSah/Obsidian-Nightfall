---
layout: default
title: Meine GitHub Repositories
---

## GitHub Repositories

<div class="container">
    <div class="row" id="repo-cards" data-masonry='{"percentPosition": true }'>
        <!-- Dynamisch generierte Repository-Karten werden hier eingefügt -->
    </div>
</div>

<div id="pagination" class="text-center">
  <button id="prev" class="btn btn-primary" disabled>Vorherige</button>
  <span id="page-info"></span>
  <button id="next" class="btn btn-primary">Nächste</button>
</div>

<script>
let currentPage = 1;
const perPage = 6;
let totalRepos = 0;

function fetchRepos(page) {
  fetch(`https://api.github.com/users/volkansah/repos?type=owner&sort=updated&per_page=${perPage}&page=${page}`)
    .then(response => {
      totalRepos = parseInt(response.headers.get('X-Total-Count') || '0');
      return response.json();
    })
    .then(data => {
      const repoContainer = document.getElementById('repo-cards');
      repoContainer.innerHTML = '';  // Leere das Container-Element

      data.filter(repo => {
        // Ausschließen von geforkten Repos und spezifischen Repos
        return !repo.fork && 
               repo.name !== 'volkansah.github.io' && 
               repo.name !== 'VolkanSah';
      }).forEach(repo => {
        // Karte erstellen
        const card = document.createElement('div');
        card.className = 'col-md-4';
        card.innerHTML = `
          <div class="card mb-4">
              <div class="card-body">
                  <h2 class="card-title">${repo.name}</h2>
                  <p class="card-text">${repo.description || 'Keine Beschreibung verfügbar'}</p>
                  <button type="button" class="btn btn-outline-primary" data-toggle="modal" data-target="#modal-${repo.name}">Mehr erfahren</button>
              </div>
          </div>
          <!-- Modal für das Repository -->
          <div class="modal fade" id="modal-${repo.name}" tabindex="-1" role="dialog" aria-labelledby="modalLabel-${repo.name}" aria-hidden="true">
              <div class="modal-dialog modal-lg">
                  <div class="modal-content">
                      <div class="modal-header">
                          <h3 class="modal-title" id="modalLabel-${repo.name}">${repo.name}</h3>
                          <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                              <span aria-hidden="true">&times;</span>
                          </button>
                      </div>
                      <div class="modal-body">
                          <p>Loading...</p>
                      </div>
                      <div class="modal-footer">
                          <button type="button" class="btn btn-primary" data-dismiss="modal">Schließen</button>
                      </div>
                  </div>
              </div>
          </div>
        `;
        repoContainer.appendChild(card);

        // README-Datei laden und in das Modal einfügen
        fetch(`https://api.github.com/repos/volkansah/${repo.name}/readme`, {
          headers: { 'Accept': 'application/vnd.github.v3.html+json' }
        })
        .then(response => response.text())
        .then(readme => {
          document.querySelector(`#modal-${repo.name} .modal-body`).innerHTML = readme;
        })
        .catch(() => {
          document.querySelector(`#modal-${repo.name} .modal-body`).innerHTML = 'Keine README-Datei gefunden.';
        });
      });

      // Masonry-Layout nach dem Hinzufügen der Karten neu laden
      const msnry = new Masonry(repoContainer, {
          itemSelector: '.col-md-4',
          percentPosition: true
      });

      updatePagination();
    })
    .catch(error => {
      console.error('Error:', error);
      const repoContainer = document.getElementById('repo-cards');
      repoContainer.innerHTML = '<p>Fehler beim Laden der Repositories.</p>';
    });
}

function updatePagination() {
  const totalPages = Math.ceil(totalRepos / perPage);
  document.getElementById('page-info').textContent = `Seite ${currentPage} von ${totalPages}`;
  document.getElementById('prev').disabled = currentPage === 1;
  document.getElementById('next').disabled = currentPage === totalPages;
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
