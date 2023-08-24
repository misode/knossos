<template>
  <Modal ref="modal" header="Create a project">
    <div v-if="!importing" class="modal-creation universal-labels">
      <div class="markdown-body">
        <p>New projects are created as drafts and can be found under your profile page.</p>
      </div>
      <label for="project-type">
        <span class="label__title">Project type<span class="required">*</span></span>
      </label>
      <Chips
        id="project-type"
        v-model="projectType"
        :items="tags.projectTypes.map((x) => x.display)"
      />
      <label for="name">
        <span class="label__title">Name<span class="required">*</span></span>
      </label>
      <input
        id="name"
        v-model="name"
        type="text"
        maxlength="64"
        placeholder="Enter project name..."
        autocomplete="off"
        @input="updatedName()"
      />
      <label for="slug">
        <span class="label__title">URL<span class="required">*</span></span>
      </label>
      <div class="text-input-wrapper">
        <div class="text-input-wrapper__before">
          https://modrinth.com/{{ getProjectType() ? getProjectType().id : '???' }}/
        </div>
        <input
          id="slug"
          v-model="slug"
          type="text"
          maxlength="64"
          autocomplete="off"
          @input="manualSlug = true"
        />
      </div>
      <label for="additional-information">
        <span class="label__title">Summary<span class="required">*</span></span>
        <span class="label__description"
          >This appears in search and on the sidebar of your project's page.</span
        >
      </label>
      <div class="textarea-wrapper">
        <textarea id="additional-information" v-model="description" maxlength="256" />
      </div>
      <div class="push-right input-group">
        <div class="external-btn">
          <button class="iconified-button" @click="beginImport">
            <LogInIcon />
            Import from external
          </button>
        </div>
        <button class="iconified-button" @click="cancel">
          <CrossIcon />
          Cancel
        </button>
        <button class="iconified-button brand-button" @click="createProject">
          <CheckIcon />
          Continue
        </button>
      </div>
    </div>
    <div v-else>
      <div v-if="page === 1" class="modal-import universal-labels">
        <div class="markdown-body">
          <p>
            You can import your project from CurseForge, GitHub, or Modrinth. If you don't have a
            project on any of these platforms, you can create a new project manually.
          </p>
        </div>
        <div class="iconified-input">
          <SearchIcon/>
          <input
            v-model="search"
            type="text"
            placeholder="Search for a project..."
            autocomplete="off"
            @input="searchProjects"
          />
        </div>
        <div v-if="selectedProject" class="project-card">
          <Avatar :src="selectedProject.icon" size="md" />
          <div class="project-text">
            <div class="title">{{selectedProject.title}}</div>
            <div class="description">{{selectedProject.description}}</div>
          </div>
        </div>
        <div v-else class="project-card">
          <div class="project-text">
            <div class="title">No project selected</div>
            <div class="description">Search for a project to import</div>
          </div>
        </div>
        <div class="input-group push-right">
          <button class="iconified-button" @click="cancel">
            <CrossIcon />
            Cancel
          </button>
          <button class="iconified-button brand-button" @click="nextPage">
            <CheckIcon />
            Continue
          </button>
        </div>
      </div>
      <div v-else-if="page === 2" class="modal-import universal-labels">
        <div class="markdown-body">
          <p>
            You can import your project from CurseForge, GitHub, or Modrinth. If you don't have a
            project on any of these platforms, you can create a new project manually.
          </p>
        </div>
        <div class="dependency-map">
          <div class="entry">
            <div class="root-project">
              <Avatar src="null" size="sm" />
              <div class="title">
                Spirit
              </div>
            </div>
            <RightArrowIcon/>
            <div class="translated-project">
              <div class="iconified-input">
                <SearchIcon/>
                <input
                  v-model="search"
                  type="text"
                  placeholder="Search for a project..."
                  autocomplete="off"
                  @input="searchProjects"
                />
              </div>
              <div v-if="false" class="mini-project">
                <Avatar src="null" size="xs" />
                <div class="text">
                  Spirit
                </div>
              </div>
              <div v-else class="mini-project">
                <div class="text">
                  No project selected
                </div>
              </div>
            </div>
          </div>
        </div>
        <div class="input-group push-right">
          <button class="iconified-button" @click="cancel">
            <CrossIcon />
            Cancel
          </button>
          <button class="iconified-button brand-button" @click="nextPage">
            <CheckIcon />
            Continue
          </button>
        </div>
      </div>
      <div v-else-if="page === 3" class="modal-import universal-labels">

      </div>
    </div>
  </Modal>
</template>

<script>
import CrossIcon from '~/assets/images/utils/x.svg'
import CheckIcon from '~/assets/images/utils/right-arrow.svg'
import { LogInIcon, SearchIcon, Avatar, RightArrowIcon } from 'omorphia'
import Modal from '~/components/ui/Modal.vue'
import Chips from '~/components/ui/Chips.vue'

export default {
  components: {
    Chips,
    CrossIcon,
    CheckIcon,
    Modal,
    LogInIcon,
    SearchIcon,
    Avatar,
    RightArrowIcon
  },
  props: {
    itemType: {
      type: String,
      default: '',
    },
    itemId: {
      type: String,
      default: '',
    },
  },
  setup() {
    const tags = useTags()

    return { tags }
  },
  data() {
    return {
      projectType: this.tags.projectTypes[0].display,
      name: '',
      slug: '',
      description: '',
      manualSlug: false,
      importing: false,
      page: 1,
      selectedProject: null
    }
  },
  methods: {
    cancel() {
      this.$refs.modal.hide()
    },
    getProjectType() {
      return this.tags.projectTypes.find((x) => this.projectType === x.display)
    },
    getClientSide() {
      switch (this.getProjectType().id) {
        case 'plugin':
          return 'unsupported'
        case 'resourcepack':
          return 'required'
        case 'shader':
          return 'required'
        case 'datapack':
          return 'optional'
        default:
          return 'unknown'
      }
    },
    getServerSide() {
      switch (this.getProjectType().id) {
        case 'plugin':
          return 'required'
        case 'resourcepack':
          return 'unsupported'
        case 'shader':
          return 'unsupported'
        case 'datapack':
          return 'required'
        default:
          return 'unknown'
      }
    },
    async createProject() {
      startLoading()

      const projectType = this.getProjectType()

      const formData = new FormData()

      const auth = await useAuth()

      formData.append(
        'data',
        JSON.stringify({
          title: this.name.trim(),
          project_type: projectType.actual,
          slug: this.slug,
          description: this.description.trim(),
          body: '',
          initial_versions: [],
          team_members: [
            {
              user_id: auth.value.user.id,
              name: auth.value.user.username,
              role: 'Owner',
            },
          ],
          categories: [],
          client_side: this.getClientSide(),
          server_side: this.getServerSide(),
          license_id: 'LicenseRef-Unknown',
          is_draft: true,
        })
      )

      try {
        await useBaseFetch('project', {
          method: 'POST',
          body: formData,
          headers: {
            'Content-Disposition': formData,
          },
        })

        this.$refs.modal.hide()
        await this.$router.push({
          name: 'type-id',
          params: {
            type: projectType.id,
            id: this.slug,
          },
          state: {
            overrideProjectType: projectType.id,
          },
        })
      } catch (err) {
        this.$notify({
          group: 'main',
          title: 'An error occurred',
          text: err.data.description,
          type: 'error',
        })
      }
      stopLoading()
    },
    show() {
      this.projectType = this.tags.projectTypes[0].display
      this.name = ''
      this.slug = ''
      this.description = ''
      this.manualSlug = false
      this.$refs.modal.show()
      this.importing = false
      this.page = 1
    },
    updatedName() {
      if (!this.manualSlug) {
        this.slug = this.name
          .trim()
          .toLowerCase()
          .replaceAll(' ', '-')
          .replaceAll(/[^a-zA-Z0-9!@$()`.+,_"-]/g, '')
          .replaceAll(/--+/gm, '-')
      }
    },
    beginImport() {
      this.importing = true
    },
    nextPage() {
      this.page++
    }
  },
}
</script>

<style scoped lang="scss">
.modal-creation {
  padding: var(--spacing-card-bg);
  display: flex;
  flex-direction: column;

  .markdown-body {
    margin-bottom: 0.5rem;
  }

  input {
    width: 20rem;
    max-width: 100%;
  }

  .text-input-wrapper {
    width: 100%;
  }

  textarea {
    min-height: 5rem;
  }

  .input-group {
    margin-top: var(--spacing-card-md);

    .external-btn {
      flex-grow: 1;
    }
  }
}

.project-card {
  display: flex;
  align-items: center;
  width: 100%;
  padding: var(--gap-md);
  border: 1px solid var(--color-button-bg);
  border-radius: var(--radius-lg);

  .project-text {
    margin-left: var(--spacing-card-md);
    display: flex;
    flex-direction: column;
    justify-content: center;

    .title {
      font-size: var(--font-size-xl);
      color: var(--color-contrast);
      font-weight: bold;
    }

    .description {
      color: var(--color-text-secondary);
    }
  }
}

.modal-import {
  padding: var(--spacing-card-bg);
  display: flex;
  flex-direction: column;
  gap: var(--gap-md);

  .text-input-wrapper {
    width: 100%;
  }

  textarea {
    min-height: 5rem;
  }
}

.root-project {
  display: flex;
  align-items: center;
  width: 50%;
  padding: var(--gap-sm);
  border: 1px solid var(--color-button-bg);
  border-radius: var(--radius-lg);
  gap: var(--gap-sm);

  :deep(.avatar) {
    --size: 4.5rem !important;
  }

  .title {
    font-size: var(--font-size-lg);
    color: var(--color-contrast);
    font-weight: bold;
  }
}

.entry {
  display: flex;
  align-items: center;
  width: 100%;
  gap: var(--gap-md);

  .entry-text {
    margin-left: var(--spacing-card-md);
    display: flex;
    flex-direction: column;
    justify-content: center;

    .title {
      font-size: var(--font-size-xl);
      color: var(--color-contrast);
      font-weight: bold;
    }

    .description {
      color: var(--color-text-secondary);
    }
  }
}

.translated-project {
  display: flex;
  flex-direction: column;
  gap: var(--gap-sm);
}

.mini-project {
  display: flex;
  align-items: center;
  width: 100%;
  padding: var(--gap-sm) var(--gap-md);
  border: 1px solid var(--color-button-bg);
  border-radius: var(--radius-md);
  gap: var(--gap-sm);

  .title {
    font-size: var(--font-size-md);
    color: var(--color-contrast);
    font-weight: bold;
  }

  :deep(.avatar) {
    --size: 1.5rem !important;
  }
}
</style>
