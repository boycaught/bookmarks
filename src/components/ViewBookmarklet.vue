<!--
  - Copyright (c) 2020. The Nextcloud Bookmarks contributors.
  -
  - This file is licensed under the Affero General Public License version 3 or later. See the COPYING file.
  -->

<template>
	<NcContent app-name="bookmarks">
		<div class="bookmarklet">
			<h2><figure :class="loading? 'icon-loading-small' : 'icon-link'" /> {{ t('bookmarks', 'Add a bookmark') }}</h2>
			<div v-if="exists" class="bookmarklet__exists">
				{{ t('bookmarks', 'This URL is already bookmarked! Overwrite?') }}
			</div>
			<label>{{ t('bookmarks', 'Title') }}
				<input v-model="bookmark.title" type="text" :placeholder="t('bookmarks', 'Enter bookmark title')">
			</label>
			<label>{{ t('bookmarks', 'Link') }}
				<input v-model="bookmark.url" type="text" :placeholder="t('bookmarks', 'Enter bookmark URL')">
			</label>
			<label><figure class="icon-tag" /> {{ t('bookmarks', 'Tags') }}
				<NcMultiselect class="sidebar__tags"
					:value="bookmark.tags"
					:auto-limit="false"
					:limit="7"
					:options="allTags"
					:multiple="true"
					:taggable="true"
					@input="onTagsChange"
					@tag="onAddTag" />
			</label>
			<label><figure class="icon-folder" /> {{ t('bookmarks', 'Folder') }}
				<input :value="folderTitle"
					type="text"
					readonly
					:placeholder="t('bookmarks', 'Root Folder')"
					@click="showPicker = true">
				<FolderPickerDialog v-model="folder" :show="showPicker" @close="showPicker = false" />
			</label>
			<label>{{ t('bookmarks', 'Notes') }}</label>
			<div class="bookmarklet__notes"
				contenteditable
				@input="onNotesChange">
				{{ description }}
			</div>
			<button class="primary" @click="submit">
				<span class="icon-confirm-white" />{{ t('bookmarks', 'Save') }}
			</button>
		</div>
	</NcContent>
</template>

<script>
import { NcContent, NcMultiselect } from '@nextcloud/vue'
import { actions } from '../store/index.js'
import FolderPickerDialog from './FolderPickerDialog.vue'

export default {
	name: 'ViewBookmarklet',
	components: {
		FolderPickerDialog,
		NcContent,
		NcMultiselect,
	},
	props: {
		title: {
			type: String,
			default: '',
		},
		url: {
			type: String,
			default: '',
		},
	},
	data() {
		return {
			bookmark: {
				title: this.title,
				url: this.url,
				tags: [],
				description: '',
			},
			description: '',
			exists: false,
			loading: true,
			showPicker: false,
			folder: -1,
		}
	},
	computed: {
		allTags() {
			return this.$store.state.tags.map(tag => tag.name)
		},
		folders() {
			return this.$store.state.folders
		},
		folderTitle() {
			return this.$store.getters.getFolder(this.folder)[0].title
		},
	},
	watch: {
	  bookmark() {
		  this.description = this.bookmark.description
	  },
	},

	async created() {
		this.loading = true
		await Promise.all([
			this.reloadTags(),
			this.reloadFolders(),
		])
		await this.findBookmark(this.bookmark.url)
		this.loading = false
	},

	methods: {
		async reloadTags() {
			return this.$store.dispatch(actions.LOAD_TAGS)
		},
		async reloadFolders() {
			return this.$store.dispatch(actions.LOAD_FOLDERS)
		},
		reloadSettings() {
			this.$store.dispatch(actions.LOAD_SETTINGS)
		},
		onNotesChange(e) {
			this.bookmark.description = e.target.textContent
		},
		onTagsChange(tags) {
			this.bookmark.tags = tags
		},
		onAddTag(tag) {
			this.bookmark.tags.push(tag)
		},
		async findBookmark(url) {
			const bookmark = await this.$store.dispatch(actions.FIND_BOOKMARK, url)
			if (bookmark) {
				this.exists = true
				this.bookmark = bookmark
				if (this.bookmark.folders.length) {
					this.folder = this.bookmark.folders[0]
				}
			}
		},

		async submit() {
			this.loading = true
			if (this.folder !== -1) {
				this.bookmark.folders = [this.folder]
			}
			if (this.exists) {
				await this.$store.dispatch(actions.SAVE_BOOKMARK, this.bookmark.id)
			} else {
				await this.$store.dispatch(actions.CREATE_BOOKMARK, this.bookmark)
			}
			window.close()
		},
	},
}
</script>
<style>
.bookmarklet {
	margin: 30px auto;
	width: 70%;
}

.bookmarklet__exists {
	background: var(--color-background-dark);
	border-radius: var(--border-radius-pill);
	font-weight: bold;
	padding: 10px 20px;
}

figure[class^=icon-] {
	display: inline-block;
}

.bookmarklet label {
	margin-top: 10px;
	display: block;
}

.bookmarklet input {
	width: 100%;
	display: block;
}

.bookmarklet__notes {
	min-height: 100px !important;
	width: auto !important;
}
</style>
