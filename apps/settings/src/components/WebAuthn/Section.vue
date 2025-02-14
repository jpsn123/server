<!--
  - @copyright 2020, Roeland Jago Douma <roeland@famdouma.nl>
  -
  - @author Roeland Jago Douma <roeland@famdouma.nl>
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
  - along with this program.  If not, see <http://www.gnu.org/licenses/>.
  -->

<template>
	<div id="security-webauthn" class="section">
		<h2>{{ t('settings', 'Passwordless Authentication') }}</h2>
		<p class="settings-hint hidden-when-empty">
			{{ t('settings', 'Set up your account for passwordless authentication following the FIDO2 standard.') }}
		</p>
		<NcNoteCard v-if="devices.length === 0" type="info">
			{{ t('settings', 'No devices configured.') }}
		</NcNoteCard>
		<h3 v-else>
			{{ t('settings', 'The following devices are configured for your account:') }}
		</h3>
		<Device v-for="device in sortedDevices"
			:key="device.id"
			:name="device.name"
			@delete="deleteDevice(device.id)" />

		<NcNoteCard v-if="!hasPublicKeyCredential" type="warning">
			{{ t('settings', 'Your browser does not support WebAuthn.') }}
		</NcNoteCard>

		<AddDevice v-if="hasPublicKeyCredential"
			:is-https="isHttps"
			:is-localhost="isLocalhost"
			@added="deviceAdded" />
	</div>
</template>

<script>
import { confirmPassword } from '@nextcloud/password-confirmation'
import NcNoteCard from '@nextcloud/vue/dist/Components/NcNoteCard.js'
import '@nextcloud/password-confirmation/dist/style.css'
import sortBy from 'lodash/fp/sortBy.js'

import AddDevice from './AddDevice.vue'
import Device from './Device.vue'
import logger from '../../logger.js'
import { removeRegistration } from '../../service/WebAuthnRegistrationSerice.js'

const sortByName = sortBy('name')

export default {
	components: {
		AddDevice,
		Device,
		NcNoteCard,
	},
	props: {
		initialDevices: {
			type: Array,
			required: true,
		},
		isHttps: {
			type: Boolean,
			default: false,
		},
		isLocalhost: {
			type: Boolean,
			default: false,
		},
		hasPublicKeyCredential: {
			type: Boolean,
			default: false,
		},
	},
	data() {
		return {
			devices: this.initialDevices,
		}
	},
	computed: {
		sortedDevices() {
			return sortByName(this.devices)
		},
	},
	methods: {
		deviceAdded(device) {
			logger.debug(`adding new device to the list ${device.id}`)

			this.devices.push(device)
		},
		async deleteDevice(id) {
			logger.info(`deleting webauthn device ${id}`)

			await confirmPassword()
			await removeRegistration(id)

			this.devices = this.devices.filter(d => d.id !== id)

			logger.info(`webauthn device ${id} removed successfully`)
		},
	},
}
</script>

<style scoped>

</style>
