<script>
  import WPAPI from 'wpapi';
  import { fade } from 'svelte/transition';
  import { onMount, afterUpdate } from 'svelte';

  let dateVal;
  let dateSelect;

  let isLoading = false;

  let date = new Date();

  let previous = new Date();
  previous.setDate(date.getDate() - 1);

  let next = new Date();
  next.setDate(date.getDate() + 1);

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
    isLoading = true;

    return await wp.posts().param('after', new Date(date.toLocaleDateString())).param('before', new Date(next.toLocaleDateString())).perPage(100).get(async (error, data) => {
      if(error) {
        console.error(error);
      }

      isLoading = false;
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

  const updatePosts = async() => {
    Object.values(users).forEach((user) => {
      user.postCount = 0;
      user.postTimes = [];
    });


    getPosts().then((data) => {
      getPostsWithUsers(data);
      
      const manicured = Object.values(users).map((user) => {
        // console.log(user.postTimes)
        return {
          "Name": user.name,
          "Post Count": user.postCount,
          "Post Times": user.postTimes,
        }
      });

      userPosts = [...manicured];
    }).catch((error) => console.error(error));
  };

  const setDate = (newDate) => {
    date = new Date(newDate);
  }

  const changeDate = (event) => {
    console.log(dateVal);
    if(dateVal) {
      console.log('yep');
      let correct = new Date(dateVal);
      correct.setDate(correct.getDate() + 1);
      setDate(correct);
      previous.setDate(date.getDate() - 1);
      next.setDate(date.getDate() + 1);
      updatePosts();
    } else {
      return;
    }
    dateSelect.blur();
  };

  onMount(() => {
    setDate(new Date(Date.now()));
    updatePosts();
  });
</script>
<div class="report-control">
  <h2 class="report-header">{date.toLocaleDateString()}</h2>
  <div class="datepicker-toggle">
    <div class="datepicker-toggle-button">

    </div>
    <input type="date" bind:this={dateSelect} bind:value={dateVal} on:change={changeDate}>
  </div>
</div>
<section class="report">
  <div class="row table-header">
    <h3>Name</h3>
    <h3>Posts</h3>
    <h3>Post Times</h3>
  </div>
  <div class="table-body">
    {#if !isLoading}
      {#each userPosts as user, index}
      <div class="user row {index % 2 === 0 ? 'even' : 'odd'}">
        <h3>{user["Name"]}</h3>
        <p>{user["Post Count"]}</p>
        <details>
          <summary>Post Times</summary>
          <ol class="times">
            {#each user["Post Times"] as time}
              <li>{new Date(time).toLocaleString()}</li>
            {/each}
          </ol>
        </details>
      </div>
      {/each}
    {/if}
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
    border: 1px solid var(--dark);
  }

  .report-header {
    text-align: center;
  }

  :global(.even) {
    background: hsl(var(--light-hs), 88%);
  }
  :global(.odd) {
    /* light l = 98% */
    background: var(--light);
  }
  .row {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 2rem;
    padding: 0.5rem;
  }

  .row > *:nth-child(2) {
    text-align: center;
  }

  .row > *:nth-child(3) {
    text-align: right;
  }
  .table-header {
    border-bottom: 1px solid var(--dark)
  }

  .report-control {
    display: flex;
    flex-direction: row;
    align-items: center;
    gap: 0.5rem;
  }

  .datepicker-toggle {
    color: var(--dark);
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
    color: var(--dark);
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
    color: #fff;
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
    /* border-bottom: 1px solid var(--dark); */
  }

  @media (prefers-color-scheme: dark) {
    .report {
      border-color: hsl(var(--light-hs), 50%);
    }
    :global(.even) {
      background: hsl(var(--dark-hs), 15%);
    }
    :global(.odd) {
      background: var(--dark);
    }
    .table-header {
      border-color: hsl(var(--light-hs), 50%);
    }
    .datepicker-toggle-button {
      color: var(--light);
    }
    .times li {
      border-color: hsl(var(--light-hs), 50%);
    }
  }
</style>