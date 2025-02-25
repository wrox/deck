<!--
  - @copyright Copyright (c) 2018 Julius Härtl <jus@bitgrid.net>
  -
  - @author Julius Härtl <jus@bitgrid.net>
  -
  - @license GNU AGPL version 3 or any later version
  -
  - This program is free software: you can redistribute it and/or modify
  - it under the terms of the GNU Affero General Public License as
  - published by the Free Software Foundation, either version 3 of the
  - License, or (at your option) any later version.
  -
  - This program is distributed in the hope that it will be useful,
  - but WITHOUT ANY WARRANTY; without even the implied warranty of
  - MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
  - GNU Affero General Public License for more details.
  -
  - You should have received a copy of the GNU Affero General Public License
  - along with this program. If not, see <http://www.gnu.org/licenses/>.
  -
  -->

<template>
	<AppSidebar v-if="currentBoard && currentCard"
		:title="title"
		:subtitle="subtitle"
		:title-editable="titleEditable"
		@update:titleEditable="handleUpdateTitleEditable"
		@update:title="handleUpdateTitle"
		@submit-title="handleSubmitTitle"
		@close="closeSidebar">
		<template #secondary-actions>
			<ActionButton v-if="cardDetailsInModal" icon="icon-menu-sidebar" @click.stop="showModal()">
				{{ t('deck', 'Open in sidebar view') }}
			</ActionButton>

			<ActionButton v-else icon="icon-external" @click.stop="showModal()">
				{{ t('deck', 'Open in bigger view') }}
			</ActionButton>
		</template>

		<AppSidebarTab id="details"
			:order="0"
			:name="t('deck', 'Details')"
			icon="icon-home">
			<CardSidebarTabDetails :card="currentCard" />
		</AppSidebarTab>

		<AppSidebarTab id="attachments"
			:order="1"
			:name="t('deck', 'Attachments')"
			icon="icon-attach">
			<CardSidebarTabAttachments :card="currentCard" />
		</AppSidebarTab>

		<AppSidebarTab
			id="comments"
			:order="2"
			:name="t('deck', 'Comments')"
			icon="icon-comment">
			<CardSidebarTabComments :card="currentCard" />
		</AppSidebarTab>

		<AppSidebarTab v-if="hasActivity"
			id="timeline"
			:order="3"
			:name="t('deck', 'Timeline')"
			icon="icon-activity">
			<CardSidebarTabActivity :card="currentCard" />
		</AppSidebarTab>
	</AppSidebar>
</template>

<script>
import { ActionButton, AppSidebar, AppSidebarTab } from '@nextcloud/vue'
import { mapState, mapGetters } from 'vuex'
import CardSidebarTabDetails from './CardSidebarTabDetails'
import CardSidebarTabAttachments from './CardSidebarTabAttachments'
import CardSidebarTabComments from './CardSidebarTabComments'
import CardSidebarTabActivity from './CardSidebarTabActivity'
import relativeDate from '../../mixins/relativeDate'

import { showError } from '@nextcloud/dialogs'

const capabilities = window.OC.getCapabilities()

export default {
	name: 'CardSidebar',
	components: {
		AppSidebar,
		AppSidebarTab,
		ActionButton,
		CardSidebarTabAttachments,
		CardSidebarTabComments,
		CardSidebarTabActivity,
		CardSidebarTabDetails,
	},
	mixins: [relativeDate],
	props: {
		id: {
			type: Number,
			required: true,
		},
	},
	data() {
		return {
			titleEditable: false,
			titleEditing: '',
			hasActivity: capabilities && capabilities.activity,
		}
	},
	computed: {
		...mapState({
			currentBoard: state => state.currentBoard,
			cardDetailsInModal: state => state.cardDetailsInModal,
		}),
		...mapGetters(['canEdit', 'assignables']),
		title() {
			return this.titleEditable ? this.titleEditing : this.currentCard.title
		},
		currentCard() {
			return this.$store.getters.cardById(this.id)
		},
		subtitle() {
			return t('deck', 'Modified') + ': ' + this.relativeDate(this.currentCard.lastModified * 1000) + ' ' + t('deck', 'Created') + ': ' + this.relativeDate(this.currentCard.createdAt * 1000)
		},
	},
	methods: {
		handleUpdateTitleEditable(value) {
			this.titleEditable = value
			if (value) {
				this.titleEditing = this.currentCard.title
			}
		},
		handleUpdateTitle(value) {
			this.titleEditing = value
		},
		handleSubmitTitle(value) {
			if (value.trim === '') {
				showError(t('deck', 'The title cannot be empty.'))
				return
			}
			this.titleEditable = false
			this.$store.dispatch('updateCardTitle', { ...this.currentCard, title: this.titleEditing })
		},

		closeSidebar() {
			this.$router.push({ name: 'board' })
		},

		showModal() {
			this.$store.dispatch('setCardDetailsInModal', true)
		},
	},
}
</script>

<style lang="scss" scoped>

	// FIXME: Obivously we should at some point not randomly reuse the sidebar component
	// since this is not oficially supported
	.modal__card .app-sidebar {
		$modal-padding: 14px;
		border: 0;
		min-width: calc(100% - #{$modal-padding*2});
		position: relative;
		top: 0;
		left: 0;
		right: 0;
		max-width: calc(100% - #{$modal-padding*2});
		padding: 14px;
		max-height: calc(100% - #{$modal-padding*2});
		&::v-deep {
			.app-sidebar-header {
				position: sticky;
				top: 0;
				z-index: 100;
				background-color: var(--color-main-background);
			}
			.app-sidebar-tabs__nav {
				position: sticky;
				top: 87px;
				margin: 0;
				z-index: 100;
				background-color: var(--color-main-background);
			}

			section {
				min-height: auto;
				display: flex;
				flex-direction: column;
			}

			#emptycontent, .emptycontent {
				margin-top: 88px;
			}
		}
	}

</style>
