<template>
	<header>
		<h1>curd app graphql-vue (vue-apollo)</h1>
	</header>
	<br />
	<main>
		<input v-model="search" placeholder="search by user name..." type="search" />

		<form v-if="search.length === 0" @submit.prevent="handleSubmit">
			<input v-model="form.name" placeholder="enter name" type="text" name="username" />
			<br />
			<input v-model="form.country" placeholder="enter country" type="text" name="country" />
			<br />
			<button type="submit">{{ form.id ? 'update' : 'submit' }}</button>
		</form>
		<br />
		Total ({{ allUsers.length }})
		<ul>
			<li v-for="user in allUsers" :key="user.id">
				<span :class="{match: user.name.indexOf(search) >= 0 && search.length}">{{ user.name }}</span> ({{ user.country }})
				<u @click="handleEditUser(user)">edit</u>
				<u @click="handleDelete(user.id)">delete</u>
			</li>
		</ul>
	</main>
</template>

<script>
import gql from 'graphql-tag'
export default {
	data() {
		return {
			search: '',
			form: {
				name: '',
				country: '',
				reset() {
					this.name = ''
					this.country = ''
				},
			},
			allUsers: [],
		}
	},
	methods: {
		async handleSearch() {
			const searchString = this.search
			const res = await this.$apollo.query({
				query: gql`
					query searchUser($search: String!) {
						searchUser(search: $search) {
							id
							name
							country
							createdAt
						}
					}
				`,
				variables: {
					search: searchString,
				},
			})
			this.allUsers = res.data.searchUser
		},
		handleEditUser(user) {
			this.form.name = user.name
			this.form.country = user.country
			this.form.id = user.id
		},
		async handleUpdate() {
			console.log('handleUpdate')
			const payload = this.form
			const res = await this.$apollo.mutate({
				mutation: gql`
					mutation updateUser($data: updateUserInput!) {
						updateUser(data: $data) {
							id
							name
							country
							createdAt
						}
					}
				`,
				variables: {
					data: payload,
				},
			})

			const index = this.allUsers.findIndex((usr) => usr.id === payload.id)
			if (index >= 0) {
				this.allUsers[index] = res.data.updateUser
			}
			this.form.reset()
		},
		async handleSubmit() {
			if (this.form.id) {
				this.handleUpdate()
				return
			}
			const payload = this.form
			const res = await this.$apollo.mutate({
				mutation: gql`
					mutation createUser($data: createUserInput!) {
						createUser(data: $data) {
							id
							name
							country
							createdAt
						}
					}
				`,
				variables: {
					data: payload,
				},
			})
			this.allUsers = [...this.allUsers, res.data.createUser]
			this.form.reset()
		},
		async handleDelete(id) {
			const res = await this.$apollo.mutate({
				mutation: gql`
					mutation deleteUser($id: String!) {
						deleteUser(id: $id) {
							id
							name
							country
							createdAt
						}
					}
				`,
				variables: {
					id: id,
				},
			})

			this.allUsers = this.allUsers.filter((user) => user.id !== id)
		},
	},
	apollo: {
		allUsers: {
			query: gql`
				query {
					getAllUsers {
						id
						name
						country
						createdAt
					}
				}
			`,
			update: function (data) {
				return data.getAllUsers
			},
		},
	},
	watch: {
		search: function (v) {
			this.handleSearch(v)
		},
	},
}
</script>


<style>

		input {
			font-size: 22px;
			margin: 5px 0px;
			border-radius: 5px;
			outline: none;
		}

	.match {
		background: yellow;
	}
</style>