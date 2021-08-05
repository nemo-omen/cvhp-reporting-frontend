<script>
  import WPAPI from 'wpapi';
  import { fade } from 'svelte/transition';
  import { onMount } from 'svelte';
  import DatepickerButton from './DatepickerButton.svelte';

  let date = new Date(Date.now()).toLocaleDateString();
  let yesterday = new Date('08/04/2021').toLocaleDateString();
  let endDate = new Date(Date.now()).toLocaleDateString();
  date = yesterday;

  const wp = new WPAPI({
    endpoint: 'https://www.conchovalleyhomepage.com/wp-json'
  });

  const users = {
    3364: {
      name: 'James Smith',
      id: 3364,
      postCount: 0,
      postTimes: [],
    },
    4800: {
      name: 'Brittany Lawrence',
      id: 4800,
      postCount: 0,
      postTimes: [],
    },
    4519: {
      name: 'Kris Boone',
      id: 4519,
      postCount: 0,
      postTimes: [],
    },
    3477: {
      name: 'Senora Scott',
      id: 3477,
      postCount: 0,
      postTimes: [],
    },
    2308: {
      name: 'Jeff Caldwell',
      id: 2308,
      postCount: 0,
      postTimes: [],
    }
  };

  $: userPosts = [];

  const getPosts = async () => {
    return await wp.posts().param('after', new Date(date)).param('before', new Date(endDate)).perPage(100).get(async (error, data) => {
      if(error) {
        console.error(error);
      }

      return await data;
    });
  };


  const getPostsWithUsers = (postData) => {
    postData.forEach((post) => {
      if(users[post.author]) {
        const postInfo = {
          authorName: users[post.author],
          date: new Date(post.date).toLocaleString(),
        }
        users[post.author].postCount ++;
        users[post.author].postTimes.push(postInfo.date);
        users[post.author].postTimes = users[post.author].postTimes.sort((a, b) => new Date(a) < new Date(b) ? -1 : 1);
      }
    });
  };

  onMount(() => {
    getPosts().then((data) => {
      getPostsWithUsers(data);
      
      const manicured = Object.values(users).map((user) => {
        return {
          "Name": user.name,
          "Post Count": user.postCount
        }
      });

      userPosts = [...manicured];
      console.log(userPosts);
    });
  });
</script>
<div class="report-control">
  <div class="datepicker">
    <input type="date">
    <button>
      <svg class="icon" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24">
          <path fill="none" d="M0 0h24v24H0z"/><path d="M17 3h4a1 1 0 0 1 1 1v16a1 1 0 0 1-1 1H3a1 1 0 0 1-1-1V4a1 1 0 0 1 1-1h4V1h2v2h6V1h2v2zm-2 2H9v2H7V5H4v4h16V5h-3v2h-2V5zm5 6H4v8h16v-8z"/>
      </svg>
    </button>
  </div>
  <h2 class="report-header">{date}</h2>
</div>
<section class="report">
  <div class="row table-header">
    <h3>Name</h3>
    <h3>Posts</h3>
  </div>
  <div class="table-body">
    {#each userPosts as user, index}
    <div class="user row {index % 2 === 0 ? 'even' : 'odd'}">
      <h3>{user["Name"]}</h3>
      <p>{user["Post Count"]}</p>
    </div>
    {/each}
  </div>
</section>

<style>
  .report-control {
    display: flex;
    justify-content: space-between;
  }

  .report {
    margin-top: 1rem;
    border: 1px solid var(--color-light);
  }

  .report-header {
    text-align: center;
  }

  :global(.even) {
    background: hsl(var(--dark-hs), 15%);
  }
  :global(.odd) {
    background: var(--dark);
  }
  .row {
    display: flex;
    justify-content: space-between;
    gap: 2rem;
    padding: 0.5rem;
  }
  .table-header {
    border-bottom: 1px solid var(--color-light)
  }

  .report-control {
    display: flex;
    flex-direction: row;
    align-items: center;
    gap: 0.5rem;
  }

  .datepicker {
    display: flex;
    align-items: center;
  }

  input[type=date] {
    background: var(--color-dark);
    color: #fff;
    border: 1px solid var(--color-light);
    padding: 0.5rem;
  }

  button {
    color: #fff;
    background: transparent;
    border: none;
    padding: 0.5rem;
  }

  .icon {
    fill: currentColor;
    width: 1rem;
    height: 1rem;
  }
</style>