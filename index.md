---
layout: default
title: My GitHub Repositories
---

<div class="container">
    <ul id="repo-list" class="list-unstyled row"></ul>
</div>

<script>
function fetchAllRepos() {
  fetch(`https://api.github.com/users/volkansah/repos?type=owner&sort=updated&per_page=100`)
    .then(response => response.json())
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
              <p class="card-text">${repo.description || 'No description available'}</p>
              <button class="btn btn-primary" data-toggle="modal" data-target="#repoModal-${index}" onclick="loadReadme('${repo.full_name}', ${index})">View Details</button>
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
                <div class="modal-body" id="repoContent-${index}">
                  <p>Loading README...</p>
                </div>
                <div class="modal-footer">
                  <a href="${repo.html_url}" target="_blank" class="btn btn-primary">Go to Repository</a>
                  <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
                </div>
              </div>
            </div>
          </div>
        `;
        repoList.appendChild(listItem);
      });
    })
    .catch(error => {
      console.error('Error:', error);
      let repoList = document.getElementById('repo-list');
      repoList.innerHTML = '<li>Error loading repositories.</li>';
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
    document.getElementById(`repoContent-${index}`).innerHTML = '<p>README could not be loaded.</p>';
  });
}

// Load all repositories on page load
fetchAllRepos();
</script>
