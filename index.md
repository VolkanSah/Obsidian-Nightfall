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
      let listItem = document.createElement('li');
      listItem.innerHTML = `<a href="${repo.html_url}">${repo.name}</a> - ${repo.description} (Forks: ${repo.forks_count})`;
      repoList.appendChild(listItem);

      // Fetch and list forks if any
      if (repo.forks_count > 0) {
        fetch(repo.forks_url)
          .then(forkResponse => forkResponse.json())
          .then(forks => {
            let forkList = document.createElement('ul');
            forks.forEach(fork => {
              let forkItem = document.createElement('li');
              forkItem.innerHTML = `<a href="${fork.html_url}">${fork.full_name}</a>`;
              forkList.appendChild(forkItem);
            });
            listItem.appendChild(forkList);
          });
      }
    });
  });


</script>
