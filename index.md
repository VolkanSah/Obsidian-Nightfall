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
      let repoList = document.getElementById('repo-list');
      let repoModals = document.getElementById('repo-modals');
      repoList.innerHTML = '';
      repoModals.innerHTML = '';

      data.filter(repo => {
        return !repo.fork && repo.name !== 'volkansah.github.io' && repo.name !== 'VolkanSah';
      }).forEach(repo => {
        // Listeneintrag für jedes Repo erstellen
        let listItem = document.createElement('li');
        listItem.innerHTML = `<a href="#" data-toggle="modal" data-target="#modal-${repo.name}">${repo.name}</a> - ${repo.description || 'Keine Beschreibung verfügbar'}`;
        repoList.appendChild(listItem);

        // Modal für jedes Repo erstellen
        let modal = document.createElement('div');
        modal.className = 'modal fade';
        modal.id = `modal-${repo.name}`;
        modal.setAttribute('tabindex', '-1');
        modal.setAttribute('role', 'dialog');
        modal.setAttribute('aria-labelledby', `modalLabel-${repo.name}`);
        modal.setAttribute('aria-hidden', 'true');
        modal.innerHTML = `
          <div class="modal-dialog modal-lg" role="document">
            <div class="modal-content">
              <div class="modal-header">
                <h5 class="modal-title" id="modalLabel-${repo.name}">${repo.name}</h5>
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
        `;
        repoModals.appendChild(modal);

        // README-Datei in das Modal laden
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
