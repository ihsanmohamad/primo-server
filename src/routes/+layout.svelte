<script>
  import { setContext } from 'svelte'
  import '$lib/assets/reset.css'
  import { browser } from '$app/environment'
  import { goto } from '$app/navigation'
  import { registerProcessors, dropdown, stores } from '@primo-app/primo'
  import user from '../stores/user'
  import { config } from '../stores'
  import supabase from '../supabase/core'
  import { watchForAutoLogin } from '../supabase/auth'
  import { users } from '../supabase/db'
  import Modal, { show, hide } from '$lib/components/Modal.svelte'
  import * as actions from '../actions'
  import SiteButtons from '$lib/components/SiteButtons.svelte'

  const { saved } = stores

  if (browser) {
    import('../compiler/processors').then(({ html, css }) => {
      registerProcessors({ html, css })
    })
    dropdown.set([
      {
        label: 'Back to Dashboard',
        icon: 'fas fa-arrow-left',
        href: '/',
        onClick: (e) => {
          if (!$saved) {
            e.preventDefault()
            window.alert(
              `Save your site before navigating away so you don't lose your changes`
            )
          }
        },
      },
      {
        component: SiteButtons,
      },
    ])
    setContext('track', () => {})
  }

  watchForAutoLogin(async (event, session) => {
    if (event === 'SIGNED_IN') {
      const { id, email } = session.user
      const [userData] = await users.get(null, 'role, sites', email)
      if (!userData) return
      user.update((u) => ({
        ...u,
        uid: id,
        id,
        email,
        signedIn: true,
        admin: userData.role === 'admin',
        role: userData.role === 'admin' ? 'developer' : userData.role,
        sites: userData.sites,
      }))
    } else if (event === 'SIGNED_OUT') {
      user.reset()
      goto('/')
    } else if (event === 'PASSWORD_RECOVERY') {
      // passwordResetToken = session.access_token;
    } else {
      console.warn('NEW AUTH EVENT', event)
    }
  })

  $: if (!$user.signedIn) {
    show({
      id: 'AUTH',
      options: {
        disableClose: true,
      },
      props: {
        onSignIn: async () => {
          await Promise.all([
            actions.sites.initialize(),
            actions.hosts.initialize(),
          ])
          hide()
        },
      },
    })
  }
</script>

<div style:--primo-color-brand={$config.customization.color}>
  <Modal />
  <slot />
</div>

<style>
  div {
    --primo-color-brand: #35d994;
    --primo-color-brand-dark: #097548;
    --primo-color-white: white;
    --primo-color-codeblack: rgb(30, 30, 30);
    --primo-color-codeblack-opaque: rgba(30, 30, 30, 0.9);

    --primo-color-black: rgb(17, 17, 17);
    --primo-color-black-opaque: rgba(17, 17, 17, 0.9);

    --color-gray-1: rgb(245, 245, 245);
    --color-gray-2: rgb(229, 229, 229);
    --color-gray-3: rgb(212, 212, 212);
    --color-gray-4: rgb(156, 163, 175);
    --color-gray-5: rgb(115, 115, 115);
    --color-gray-6: rgb(82, 82, 82);
    --color-gray-7: rgb(64, 64, 64);
    --color-gray-8: rgb(38, 38, 38);
    --color-gray-9: rgb(23, 23, 23);

    --font-size-1: 0.75rem;
    --font-size-2: 0.875rem;
    --font-size-3: 1.125rem;
    --font-size-4: 1.25rem;

    --box-shadow-xl: 0 0 #0000, 0 0 #0000, 0 20px 25px -5px rgba(0, 0, 0, 0.1),
      0 10px 10px -5px rgba(0, 0, 0, 0.04);

    --transition-colors: background-color 0.1s, border-color 0.1s, color 0.1s,
      fill 0.1s, stroke 0.1s;

    --padding-container: 15px;
    --max-width-container: 1900px;

    --ring: 0px 0px 0px 2px var(--primo-color-brand);

    --primo-max-width-1: 30rem;
    --primo-max-width-2: 1200px;
    --primo-max-width-max: 1200px;

    --primo-border-radius: 5px;
    --primo-ring-brand: 0px 0px 0px 2px var(--primo-color-brand);
    --primo-ring-brand-thin: 0px 0px 0px 1px var(--primo-color-brand);
    --primo-ring-brand-thick: 0px 0px 0px 3px var(--primo-color-brand);
  }
</style>
