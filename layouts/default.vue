<template>
  <v-app dark>
    <v-navigation-drawer
      v-model="drawer"
      fixed
      app
    >
      <v-list>
        <v-list-item
          v-for="(item, i) in items"
          :key="i"
          :to="item.to"
          router
          exact
        >
          <v-list-item-action>
            <v-icon>{{ item.icon }}</v-icon>
          </v-list-item-action>
          <v-list-item-content>
            <v-list-item-title v-text="item.title" />
          </v-list-item-content>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>
    <v-app-bar
      fixed
      app
    >
      <v-app-bar-nav-icon @click.stop="drawer = !drawer" />
      <v-btn
        icon
        @click.stop="miniVariant = !miniVariant"
      >
        <v-icon>mdi-{{ `chevron-${miniVariant ? 'right' : 'left'}` }}</v-icon>
      </v-btn>
      <v-toolbar-title v-text="title" />
      <v-spacer />
    </v-app-bar>
    <v-main>
      <v-container>
        <Nuxt />
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
export default {
  data () {
    return {
      drawer: true,
      fixed: false,
      items: [
        {
          icon: 'mdi-apps',
          title: 'Dashboard',
          to: '/admin/dashboard'
        },
        {
          icon: 'mdi-home',
          title: 'Vaccination Centres',
          to: '/admin/centres'
        },
        {
          icon: 'mdi-doctor',
          title: 'Nurses',
          to: '/admin/nurses'
        },
        {
          icon: 'mdi-clock',
          title: 'Slots',
          to: '/admin/slots'
        },
        {
          icon: 'mdi-calendar',
          title: 'Reservations',
          to: '/admin/reservations'
        }
      ],
      miniVariant: false,
      left: true,
      leftDrawer: true,
      title: 'Homage'
    }
  }
}
</script>
