---
layout: default
title: Meine GitHub Repositories
---


<div class="container">
    <div class="row" id="repo-cards" data-masonry='{"percentPosition": true }'>
        <!-- Dynamisch generierte Repository-Karten werden hier eingefügt -->
    </div>
</div>

<div id="pagination" class="text-center mt-4">



<div id="pagination" class="text-center mt-4">
  <button id="prev" class="btn btn-primary" disabled>Vorherige</button>
  <span id="page-info" class="mx-2"></span>
  <button id="next" class="btn btn-primary">Nächste</button>
</div>

<script src="https://unpkg.com/masonry-layout@4/dist/masonry.pkgd.min.js"></script>
<script>
let currentPage = 1;
const perPage = 6;
let totalRepos = 0;
let msnry;

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
        card.className = 'col-md-4 mb-4';
        card.innerHTML = `
          <div class="card h-100">
              <div class="card-body">
                  <h5 class="card-title">${repo.name}</h5>
                  <p class="card-text">${repo.description || 'Keine Beschreibung verfügbar'}</p>
                  <button type="button" class="btn btn-outline-primary" data-bs-toggle="modal" data-bs-target="#modal-${repo.name}">Mehr erfahren</button>
              </div>
          </div>
          <!-- Modal für das Repository -->
          <div class="modal fade" id="modal-${repo.name}" tabindex="-1" aria-labelledby="modalLabel-${repo.name}" aria-hidden="true">
              <div class="modal-dialog modal-lg">
                  <div class="modal-content">
                      <div class="modal-header">
                          <h5 class="modal-title" id="modalLabel-${repo.name}">${repo.name}</h5>
                          <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                      </div>
                      <div class="modal-body">
                          <p>Loading...</p>
                      </div>
                      <div class="modal-footer">
                          <button type="button" class="btn btn-primary" data-bs-dismiss="modal">Schließen</button>
                      </div>
                  </div>
              </div>
          </div>
        `;
        repoContainer.appendChild(card);

        // README-Datei laden und in das Modal einfügen
        fetch(`https://api.github.com/repos/volkansah/${repo.name}/readme`, {
          headers: { 'Accept': 'application/vnd.github.v3.html' }
        })
        .then(response => response.text())
        .then(readme => {
          document.querySelector(`#modal-${repo.name} .modal-body`).innerHTML = readme;
        })
        .catch(() => {
          document.querySelector(`#modal-${repo.name} .modal-body`).innerHTML = 'Keine README-Datei gefunden.';
        });
      });

      // Masonry-Layout nach dem Hinzufügen der Karten initialisieren oder aktualisieren
      if (msnry) {
        msnry.reloadItems();
        msnry.layout();
      } else {
        msnry = new Masonry(repoContainer, {
          itemSelector: '.col-md-4',
          percentPosition: true
        });
      }

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
