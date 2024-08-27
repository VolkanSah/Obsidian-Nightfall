---
layout: default
title: Meine GitHub Repositories
---

## GitHub Repositories

<div class="container">
    <ul id="repo-list" class="list-unstyled"></ul>
    <div id="pagination" class="text-center mt-4">
        <button id="prev" class="btn btn-primary" disabled>Vorherige</button>
        <span id="page-info" class="mx-2"></span>
        <button id="next" class="btn btn-primary">Nächste</button>
    </div>
</div>

<script>
let currentPage = 1;
const perPage = 100;
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
      data.filter(repo => {
        // Ausschließen von geforkten Repos und spezifischen Repos
        return !repo.fork && 
               repo.name !== 'volkansah.github.io' && 
               repo.name !== 'VolkanSah';
      }).forEach(repo => {
        let listItem = document.createElement('li');
        listItem.innerHTML = `<a href="${repo.html_url}">${repo.name}</a> - ${repo.description || 'Keine Beschreibung verfügbar'}`;
        repoList.appendChild(listItem);
      });
      updatePagination();
    })
    .catch(error => {
      console.error('Error:', error);
      let repoList = document.getElementById('repo-list');
      repoList.innerHTML = '<li>Fehler beim Laden der Repositories.</li>';
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
