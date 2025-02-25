<!--
  - @copyright Copyright (c) 2018 Julius Härtl <jus@bitgrid.net>
  -
  - @copyright Copyright (c) 2019 Gary Kim <gary@garykim.dev>
  -
  - @author Julius Härtl <jus@bitgrid.net>
  -
  - @author Gary Kim <gary@garykim.dev>
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
	<AttachmentDragAndDrop v-if="card" :card-id="card.id" class="drop-upload--card">
		<div :class="{'compact': compactMode, 'current-card': currentCard, 'has-labels': card.labels && card.labels.length > 0, 'is-editing': editing, 'card__editable': canEdit}"
			tag="div"
			class="card"
			@click="openCard">
			<div class="card-upper">
				<h3 v-if="compactMode || isArchived || showArchived || !canEdit">
					{{ card.title }}
				</h3>
				<h3 v-else-if="!editing">
					<span @click.stop="startEditing(card)">{{ card.title }}</span>
				</h3>

				<form v-if="editing"
					v-click-outside="cancelEdit"
					class="dragDisabled"
					@click.stop
					@keyup.esc="cancelEdit"
					@submit.prevent="finishedEdit(card)">
					<input v-model="copiedCard.title"
						v-focus
						type="text"
						required
						pattern=".*\S+.*">
					<input type="submit" value="" class="icon-confirm">
				</form>

				<DueDate v-if="!editing" :card="card" />

				<CardMenu v-if="!editing && compactMode" :card="card" class="right" />
			</div>
			<transition-group v-if="card.labels && card.labels.length"
				name="zoom"
				tag="ul"
				class="labels"
				@click.stop="openCard">
				<li v-for="label in labelsSorted" :key="label.id" :style="labelStyle(label)">
					<span @click.stop="applyLabelFilter(label)">{{ label.title }}</span>
				</li>
			</transition-group>
			<div v-show="!compactMode" class="card-controls compact-item" @click="openCard">
				<CardBadges :card="card" />
			</div>
		</div>
	</AttachmentDragAndDrop>
</template>

<script>
import ClickOutside from 'vue-click-outside'
import { mapState, mapGetters } from 'vuex'
import CardBadges from './CardBadges'
import Color from '../../mixins/color'
import labelStyle from '../../mixins/labelStyle'
import AttachmentDragAndDrop from '../AttachmentDragAndDrop'
import CardMenu from './CardMenu'
import DueDate from './badges/DueDate'

export default {
	name: 'CardItem',
	components: { CardBadges, AttachmentDragAndDrop, CardMenu, DueDate },
	directives: {
		ClickOutside,
	},
	mixins: [Color, labelStyle],
	props: {
		id: {
			type: Number,
			default: null,
		},
		item: {
			type: Object,
			default: null,
		},
	},
	data() {
		return {
			editing: false,
			copiedCard: null,
		}
	},
	computed: {
		...mapState({
			compactMode: state => state.compactMode,
			showArchived: state => state.showArchived,
			currentBoard: state => state.currentBoard,
		}),
		...mapGetters([
			'isArchived',
		]),
		canEdit() {
			if (this.currentBoard) {
				return !this.currentBoard.archived && this.$store.getters.canEdit
			}
			const board = this.$store.getters.boards.find((item) => item.id === this.card.boardId)
			return board ? !board.archived && board.permissions.PERMISSION_EDIT : false
		},
		card() {
			return this.item ? this.item : this.$store.getters.cardById(this.id)
		},
		currentCard() {
			return this.card && this.$route && this.$route.params.cardId === this.card.id
		},
		labelsSorted() {
			return [...this.card.labels].sort((a, b) => (a.title < b.title) ? -1 : 1)
		},
	},
	watch: {
		currentCard(newValue) {
			if (newValue) {
				this.$nextTick(() => this.$el.scrollIntoView({ behavior: 'smooth', block: 'center', inline: 'center' }))
			}
		},
	},
	methods: {
		openCard() {
			const boardId = this.card && this.card.boardId ? this.card.boardId : this.$route.params.id
			this.$router.push({ name: 'card', params: { id: boardId, cardId: this.card.id } }).catch(() => {})
		},
		startEditing(card) {
			this.copiedCard = Object.assign({}, card)
			this.editing = true
		},
		finishedEdit(card) {
			if (this.copiedCard.title !== card.title) {
				this.$store.dispatch('updateCardTitle', this.copiedCard)
			}
			this.editing = false
		},
		cancelEdit() {
			this.editing = false
		},
		applyLabelFilter(label) {
			this.$nextTick(() => this.$store.dispatch('toggleFilter', { tags: [label.id] }))
		},
	},
}
</script>

<style lang="scss" scoped>
	@import './../../css/animations';
	@import './../../css/variables';

	.card {
		transition: box-shadow 0.1s ease-in-out;
		box-shadow: 0 0 2px 0 var(--color-box-shadow);
		border-radius: var(--border-radius-large);
		font-size: 100%;
		background-color: var(--color-main-background);
		margin-bottom: $card-spacing;

		&::v-deep * {
			cursor: pointer;
		}

		body.dark &, body.theme--dark & {
			border: 2px solid var(--color-border);
		}

		&:hover {
			box-shadow: 0 0 5px 0 var(--color-box-shadow);
		}
		&.current-card {
			box-shadow: 0 0 5px 1px var(--color-box-shadow);
		}

		.card-upper {
			display: flex;
			min-height: 44px;
			form {
				display: flex;
				padding: 3px 5px;
				width: 100%;
				input[type=text] {
					flex-grow: 1;
				}

			}
			h3 {
				margin: 12px $card-padding;
				flex-grow: 1;
				font-size: 100%;
				overflow: hidden;
				word-wrap: break-word;
				padding-left: 4px;
				span {
					cursor: text;
				}
			}
			input[type=text] {
				font-size: 100%;
			}
		}

		@import './../../css/labels';

		.card-controls {
			display: flex;
			margin-left: $card-padding;
			margin-right: $card-padding;

			& > div {
				display: flex;
				max-height: 44px;
			}
		}
		&.card__editable .card-controls {
			margin-right: 0;
		}
	}

	.duedate {
		margin-right: 9px;
	}

	.right {
		display: flex;
		align-items: flex-start;
	}

	.compact {
		min-height: 44px;

		.duedate {
			margin-right: 0;
		}
		&.has-labels {
			padding-bottom: $card-padding;
		}
		.labels {
			height: 6px;
			margin-top: -7px;
			margin-bottom: 3px;
		}
		.labels li {
			width: 30px;
			height: 6px;
			font-size: 0;
			color: transparent;
		}
	}
</style>
