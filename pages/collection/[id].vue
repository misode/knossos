<script setup>

import { Avatar, Button, Promotion } from "omorphia";
import ProjectCard from "~/components/ui/ProjectCard.vue";
</script>

<template>
<div class="collection">
  <div class="banner">
    <Avatar size="md"/>
    <div class="banner__info">
      <h1 class="banner__title">Collection Name</h1>
      <div class="banner__description markdown-body">
        <p>
          Collection Description
        </p>
      </div>
      <div class="banner__links">
        <a>Link</a>
        <a>Link</a>
        <a>Link</a>
      </div>
    </div>
    <div class="banner__buttons">
      <div class="first-row">
        <Button>Follow</Button>
        <Button>Share</Button>
      </div>
      <Button>Report</Button>
    </div>
  </div>
  <Promotion/>
  <div class="display-mode--list">
    <ProjectCard />
    <ProjectCard />
    <ProjectCard />
    <ProjectCard />
  </div>
</div>
</template>

<style scoped lang="scss">
.banner {
  display: grid;
  grid-template-columns: 6rem 1fr min-content;
  gap: var(--gap-md);
  margin-bottom: var(--gap-md);

  .banner__info {
    display: flex;
    flex-direction: column;
    justify-content: center;

    .banner__title {
      margin: 0;
    }

    .banner__links {
      display: flex;
      gap: var(--gap-sm);
    }
  }

  .banner__buttons {
    display: flex;
    flex-direction: column;
    gap: var(--gap-sm);
    align-items: flex-end;
    justify-content: center;

    .first-row {
      display: flex;
      flex-direction: row;
      gap: var(--gap-sm);
    }
  }
}

.collection {
  max-width:  1000px;
  margin: 0 auto;
}

.display-mode--list {
  display: flex;
  flex-direction: column;
  gap: var(--gap-md);
}
</style>
