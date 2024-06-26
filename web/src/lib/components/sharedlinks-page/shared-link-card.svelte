<script lang="ts">
  import Icon from '$lib/components/elements/icon.svelte';
  import { AppRoute } from '$lib/constants';
  import { SharedLinkType, type SharedLinkResponseDto } from '@immich/sdk';
  import { mdiCircleEditOutline, mdiContentCopy, mdiDelete, mdiOpenInNew } from '@mdi/js';
  import * as luxon from 'luxon';
  import { createEventDispatcher } from 'svelte';
  import CircleIconButton from '../elements/buttons/circle-icon-button.svelte';
  import { locale } from '$lib/stores/preferences.store';
  import AlbumCover from '$lib/components/album-page/album-cover.svelte';

  export let link: SharedLinkResponseDto;

  let expirationCountdown: luxon.DurationObjectUnits;
  const dispatch = createEventDispatcher<{
    delete: void;
    copy: void;
    edit: void;
  }>();

  const getCountDownExpirationDate = () => {
    if (!link.expiresAt) {
      return;
    }

    const expiresAtDate = luxon.DateTime.fromISO(new Date(link.expiresAt).toISOString(), { locale: $locale });
    const now = luxon.DateTime.now();

    expirationCountdown = expiresAtDate.diff(now, ['days', 'hours', 'minutes', 'seconds']).toObject();

    if (expirationCountdown.days && expirationCountdown.days > 0) {
      return expiresAtDate.toRelativeCalendar({ base: now, locale: 'en-US', unit: 'days' });
    } else if (expirationCountdown.hours && expirationCountdown.hours > 0) {
      return expiresAtDate.toRelativeCalendar({ base: now, locale: 'en-US', unit: 'hours' });
    } else if (expirationCountdown.minutes && expirationCountdown.minutes > 0) {
      return expiresAtDate.toRelativeCalendar({ base: now, locale: 'en-US', unit: 'minutes' });
    } else if (expirationCountdown.seconds && expirationCountdown.seconds > 0) {
      return expiresAtDate.toRelativeCalendar({ base: now, locale: 'en-US', unit: 'seconds' });
    }
  };

  const isExpired = (expiresAt: string) => {
    const now = Date.now();
    const expiration = new Date(expiresAt).getTime();

    return now > expiration;
  };
</script>

<div
  class="flex w-full gap-4 border-b border-gray-200 py-4 transition-all hover:border-immich-primary dark:border-gray-600 dark:text-immich-gray dark:hover:border-immich-dark-primary"
>
  <div>
    <AlbumCover album={link?.album} css="h-[100px] w-[100px] transition-all duration-300 hover:shadow-lg" />
  </div>

  <div class="flex flex-col justify-between">
    <div class="info-top">
      <div class="font-mono text-xs font-semibold text-gray-500 dark:text-gray-400">
        {#if link.expiresAt}
          {#if isExpired(link.expiresAt)}
            <p class="font-bold text-red-600 dark:text-red-400">Expired</p>
          {:else}
            <p>
              Expires {getCountDownExpirationDate()}
            </p>
          {/if}
        {:else}
          <p>Expires ∞</p>
        {/if}
      </div>

      <div class="text-sm">
        <div class="flex place-items-center gap-2 text-immich-primary dark:text-immich-dark-primary">
          {#if link.type === SharedLinkType.Album}
            <p>
              {link.album?.albumName.toUpperCase()}
            </p>
          {:else if link.type === SharedLinkType.Individual}
            <p>INDIVIDUAL SHARE</p>
          {/if}

          {#if !link.expiresAt || !isExpired(link.expiresAt)}
            <a href="{AppRoute.SHARE}/{link.key}" title="Go to share page">
              <Icon path={mdiOpenInNew} />
            </a>
          {/if}
        </div>

        <p class="text-sm">{link.description ?? ''}</p>
      </div>
    </div>

    <div class="info-bottom flex gap-4">
      {#if link.allowUpload}
        <div
          class="flex w-[80px] place-content-center place-items-center rounded-full bg-immich-primary px-2 py-1 text-xs text-white dark:bg-immich-dark-primary dark:text-immich-dark-gray"
        >
          Upload
        </div>
      {/if}

      {#if link.allowDownload}
        <div
          class="flex w-[100px] place-content-center place-items-center rounded-full bg-immich-primary px-2 py-1 text-xs text-white dark:bg-immich-dark-primary dark:text-immich-dark-gray"
        >
          Download
        </div>
      {/if}

      {#if link.showMetadata}
        <div
          class="flex w-[60px] place-content-center place-items-center rounded-full bg-immich-primary px-2 py-1 text-xs text-white dark:bg-immich-dark-primary dark:text-immich-dark-gray"
        >
          EXIF
        </div>
      {/if}

      {#if link.password}
        <div
          class="flex w-[100px] place-content-center place-items-center rounded-full bg-immich-primary px-2 py-1 text-xs text-white dark:bg-immich-dark-primary dark:text-immich-dark-gray"
        >
          Password
        </div>
      {/if}
    </div>
  </div>

  <div class="flex flex-auto flex-col place-content-center place-items-end text-right">
    <div class="flex">
      <CircleIconButton title="Delete link" icon={mdiDelete} on:click={() => dispatch('delete')} />
      <CircleIconButton title="Edit link" icon={mdiCircleEditOutline} on:click={() => dispatch('edit')} />
      <CircleIconButton title="Copy link" icon={mdiContentCopy} on:click={() => dispatch('copy')} />
    </div>
  </div>
</div>
