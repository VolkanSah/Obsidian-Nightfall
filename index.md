---
layout: default
title: My GitHub Repositories
---

<div class="container">
    <div class="row" id="repo-list" data-masonry='{"percentPosition": true }'></div>
    <div class="row mt-3">
        <div class="col-12 text-center">
            <button id="prevPage" class="btn btn-secondary" onclick="loadPrevPage()">Previous</button>
            <button id="nextPage" class="btn btn-secondary" onclick="loadNextPage()">Next</button>
        </div>
    </div>
</div>

<script>
let currentPage = 1;
const perPage = 18; // Number of repositories per page

// Function to fetch and display GitHub repositories
function fetchAllRepos(page = 1) {
    // Make sure to replace 'volkansah' with your own GitHub username
  fetch(`https://api.github.com/users/volkansah/repos?type=owner&sort=updated&per_page=${perPage}&page=${page}`)
    .then(response => {
      const linkHeader = response.headers.get('Link');
      updatePaginationButtons(linkHeader);
      return response.json();
    })
    .then(data => {
      let repoList = document.getElementById('repo-list');
      repoList.innerHTML = '';
        // Filter out repositories you don't want to display, specify the repo names
      let filteredData = data.filter(repo => {
        return !repo.fork && 
               repo.name !== 'volkansah.github.io' && 
               repo.name !== 'VolkanSah';
      });
      filteredData.forEach((repo, index) => {
        let listItem = document.createElement('div');
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
                  <h3 class="modal-title" id="repoModalLabel-${index}">Name: ${repo.name}</h3>
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
      // Reinitialize Masonry after all items are added
      imagesLoaded(repoList, function() {
        new Masonry(repoList, {
          itemSelector: '.col-md-4',
          percentPosition: true
        });
      });
    })
    .catch(error => {
      console.error('Error:', error);
      let repoList = document.getElementById('repo-list');
      repoList.innerHTML = '<li>Error loading repositories.</li>';
    });
}

// Function to update pagination buttons based on the Link header from the GitHub API response
function updatePaginationButtons(linkHeader) {
  const links = parseLinkHeader(linkHeader);
  const prevButton = document.getElementById('prevPage');
  const nextButton = document.getElementById('nextPage');

  prevButton.disabled = !links.prev;
  nextButton.disabled = !links.next;
}

// Function to parse the Link header for pagination links
function parseLinkHeader(header) {
  if (!header) return {};
  const parts = header.split(',');
  const links = {};
  parts.forEach(p => {
    const section = p.split(';');
    if (section.length != 2) return;
    const url = section[0].replace(/<(.*)>/, '$1').trim();
    const name = section[1].replace(/rel="(.*)"/, '$1').trim();
    links[name] = url;
  });
  return links;
}

// Function to load the previous page of repositories
function loadPrevPage() {
  if (currentPage > 1) {
    currentPage--;
    fetchAllRepos(currentPage);
  }
}

// Function to load the next page of repositories
function loadNextPage() {
  currentPage++;
  fetchAllRepos(currentPage);
}

// Function to load the README.md file of a repository
function loadReadme(repoFullName, index) {
  fetch(`https://api.github.com/repos/${repoFullName}/readme`, {
    headers: { 'Accept': 'application/vnd.github.v3.html' }
  })
  .then(response => response.text())
  .then(data => {
    // Adjust image URLs in README
    const repoUrl = `https://github.com/${repoFullName}/blob/master/`;
    data = data.replace(/src="([^"]+)"/g, (match, p1) => {
      // Check if the path is relative
      if (!p1.startsWith('http') && !p1.startsWith('//')) {
        return `src="${repoUrl}${p1}"`;
      }
      return match;
    });

    // Adjust anchor links in README
    data = data.replace(/href="#([^"]+)"/g, (match, p1) => {
      return `href="#repoContent-${index}-${p1}"`;
    });

    // Adjust IDs in README content
    data = data.replace(/id="([^"]+)"/g, (match, p1) => {
      return `id="repoContent-${index}-${p1}"`;
    });

    document.getElementById(`repoContent-${index}`).innerHTML = data;
  })
  .catch(error => {
    document.getElementById(`repoContent-${index}`).innerHTML = '<p>README could not be loaded.</p>';
  });
}

// Initial load of repositories
fetchAllRepos();
</script>
