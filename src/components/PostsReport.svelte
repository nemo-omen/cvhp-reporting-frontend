<script>
  import WPAPI from 'wpapi';
  import { fade } from 'svelte/transition';
  import { onMount } from 'svelte';

  let date = new Date(Date.now());

  let dayBefore = new Date();
  dayBefore.setDate(date.getDate() - 1);

  let dayAfter = new Date();
  dayAfter.setDate(date.getDate() + 1);

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
    return await wp.posts().param('after', new Date(date.toLocaleDateString())).param('before', new Date(dayAfter.toLocaleDateString())).perPage(100).get(async (error, data) => {
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

  const changeDate = (event) => {
    console.log(event.target.value);
  };

  onMount(() => {
    getPosts().then((data) => {
      getPostsWithUsers(data);
      
      const manicured = Object.values(users).map((user) => {
        console.log(user.postTimes)
        return {
          "Name": user.name,
          "Post Count": user.postCount,
          "Post Times": user.postTimes,
        }
      });

      userPosts = [...manicured];
      console.log(userPosts);
    });
  });
</script>
<div class="report-control">
  <h2 class="report-header">{date.toLocaleDateString()}</h2>
  <div class="datepicker-toggle">
    <div class="datepicker-toggle-button">

    </div>
    <input type="date" on:change={changeDate}>
  </div>
</div>
<section class="report">
  <div class="row table-header">
    <h3>Name</h3>
    <h3>Posts</h3>
    <h3>Post Times</h3>
  </div>
  <div class="table-body">
    {#each userPosts as user, index}
    <div class="user row {index % 2 === 0 ? 'even' : 'odd'}">
      <h3>{user["Name"]}</h3>
      <p>{user["Post Count"]}</p>
      <details>
        <summary>Post Times</summary>
        <ol class="times">
          {#each user["Post Times"] as time}
            <li>{time}</li>
          {/each}
        </ol>
      </details>
    </div>
    {/each}
  </div>
</section>

<style>
  .report-control {
    display: flex;
    justify-content: space-between;
    padding: 0.5rem;
  }

  .report {
    margin-top: 0.5rem;
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
    display: grid;
    grid-template-columns: 1fr 1fr 2fr;
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

  .datepicker-toggle {
    color: #fff;
    display: inline-block;
    position: relative;
    width: 24px;
    height: 24px;
  }

  
  .datepicker-toggle-button {
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    background-image: url('/images/calendar.svg');
  }
  
  input[type=date] {
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    opacity: 0;
    cursor: pointer;
    box-sizing: border-box;
    background: var(--color-dark);
    color: #fff;
    border: 1px solid var(--color-light);
    padding: 0.5rem;
  }

  input[type=date]::-webkit-calendar-picker-indicator {
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    margin: 0;
    padding: 0;
  }
  .times {
    margin: 0;
    padding: 0;
    list-style-type: none;;
  }
  .times li {
    padding: 0.5rem;
    margin: 0;
    border-bottom: 1px solid var(--color-light);
  }
</style>