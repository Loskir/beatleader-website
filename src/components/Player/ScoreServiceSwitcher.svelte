<script>
  import {createEventDispatcher} from 'svelte'
  import createScoresService from '../../services/scoresaber/scores'
  import createBeatSaviorService from '../../services/beatsavior'
  import createAccSaberService from '../../services/accsaber'
  import Switcher from '../Common/Switcher.svelte'
  import ScoreServiceFilters from './ScoreServiceFilters.svelte'
  import TextFilter from './ScoreFilters/TextFilter.svelte'
  import SelectFilter from './ScoreFilters/SelectFilter.svelte'

  export let playerId = null;
  export let service = 'scoresaber';
  export let serviceParams = {sort: 'recent', order: 'desc'}
  export let loadingService = null;
  export let loadingServiceParams = null;

  const dispatch = createEventDispatcher();

  const scoresService = createScoresService();
  const beatSaviorService = createBeatSaviorService();
  const accSaberService = createAccSaberService();

  let availableServiceNames = ['scoresaber'];
  let accSaberCategories = null;

  const allServices = [
    {
      id: 'scoresaber',
      label: 'Score Saber',
      icon: '<div class="scoresaber-icon"></div>',
      url: `/u/${playerId}/scoresaber/recent/1`,
      switcherComponents: [
        {
          component: Switcher,
          props: {
            values: [
              {id: 'recent', 'label': 'Recent', iconFa: 'fa fa-clock', url: `/u/${playerId}/scoresaber/recent/1`},
              {id: 'top', 'label': 'PP', iconFa: 'fa fa-cubes', url: `/u/${playerId}/scoresaber/top/1`},
            ],
          },
          key: 'sort',
          onChange: event => {
            if (!event?.detail?.id) return null;

            dispatch('service-params-change', {sort: event?.detail?.id})
          }
        }
      ],
    },
    {
      id: 'beatsavior',
      label: 'Beat Savior',
      icon: '<div class="beatsavior-icon"></div>',
      url: `/u/${playerId}/beatsavior/recent/1`,
      switcherComponents: [
        {
          component: Switcher,
          props: {
            values: [
              {id: 'recent', 'label': 'Recent', iconFa: 'fa fa-clock', url: `/u/${playerId}/beatsavior/recent/1`},
              {id: 'acc', 'label': 'Acc', iconFa: 'fa fa-crosshairs', url: `/u/${playerId}/beatsavior/acc/1`},
              {id: 'mistakes', 'label': 'Mistakes', iconFa: 'fa fa-times', url: `/u/${playerId}/beatsavior/mistake/1`},
            ],
          },
          key: 'sort',
          onChange: event => {
            if (!event?.detail?.id) return null;

            dispatch('service-params-change', {sort: event?.detail?.id})
          }
        },
      ],
    },
    {
      id: 'accsaber',
      label: 'AccSaber',
      icon: '<div class="accsaber-icon"></div>',
      url: `/u/${playerId}/accsaber/recent/1`,
      switcherComponents: [
        {
          component: Switcher,
          key: 'type',
          onChange: event => {
            if (!event?.detail?.id) return null;

            dispatch('service-params-change', {type: event?.detail?.id})
          },
        },
        {
          component: Switcher,
          key: 'sort',
          props: {
            values: [
              {id: 'ap', 'label': 'AP', iconFa: 'fa fa-cubes'},
              {id: 'recent', 'label': 'Recent', iconFa: 'fa fa-clock'},
              {id: 'acc', 'label': 'Acc', iconFa: 'fa fa-crosshairs'},
              {id: 'rank', 'label': 'Rank', iconFa: 'fa fa-list-ol'},
            ],
          },
          onChange: event => {
            if (!event?.detail?.id) return null;

            dispatch('service-params-change', {sort: event?.detail?.id})
          },
        },
      ],
    },
  ];

  async function updateAvailableServiceNames(playerId) {
    accSaberCategories = null;

    const additionalServices = (await Promise.all([
        scoresService.isDataForPlayerAvailable(playerId).then(r => r ? 'scoresaber-cached' : null),
        beatSaviorService.isDataForPlayerAvailable(playerId).then(r => r ? 'beatsavior' : null),
        accSaberService.isDataForPlayerAvailable(playerId).then(r => r ? 'accsaber' : null),
      ])
    ).filter(s => s);

    if (additionalServices?.length) availableServiceNames = ['scoresaber'].concat(additionalServices);

    if (additionalServices.includes('accsaber')) accSaberCategories = await accSaberService.getCategories();
  }

  function updateAvailableServices(avaiableServiceNames, service, loadingService, serviceParams, loadingServiceParams, accSaberCategories) {
    const commonFilters = [
      {
        component: TextFilter,
        props: {
          id: 'search',
          iconFa: 'fa fa-search',
          title: 'Search by song/artist/mapper name',
          placeholder: 'Enter song name...'
        }
      },
      {
        component: SelectFilter,
        props: {
          id: 'diff',
          iconFa: 'fa fa-chart-line',
          title: 'Filter by map difficulty',
          values: [
            {id: null, name: 'All'},
            {id: 'easy', name: 'Easy'},
            {id: 'normal', name: 'Normal'},
            {id: 'hard', name: 'Hard'},
            {id: 'expert', name: 'Expert'},
            {id: 'expertplus', name: 'Expert+'},
          ]
        }
      }
    ];

    return allServices
      .filter(s => availableServiceNames.includes(s?.id))
      .map(s => {
        if (s?.id !== service || !s?.switcherComponents?.length) return s;

        const serviceDef = {...s};
        serviceDef.switcherComponents = serviceDef.switcherComponents.map(c => ({...c}));

        switch (service) {
          case 'scoresaber':
            if (availableServiceNames.includes('scoresaber-cached')) {
              const sortComponent = serviceDef.switcherComponents.find(c => c.key === 'sort');
              if (sortComponent?.props?.values) {
                if (!sortComponent.props.values.find(v => v.id === 'rank'))
                  sortComponent.props.values.push({
                    id: 'rank',
                    label: 'Rank',
                    iconFa: 'fa fa-list-ol',
                    title: 'May be inaccurate - rank is from last score refresh',
                  });

                if (!sortComponent.props.values.find(v => v.id === 'acc'))
                  sortComponent.props.values.push({
                    id: 'acc',
                    label: 'Acc',
                    iconFa: 'fa fa-crosshairs',
                    title: 'Sort by accuracy',
                  });

                if (!sortComponent.props.values.find(v => v.id === 'stars'))
                  sortComponent.props.values.push({
                    id: 'stars',
                    label: 'Stars',
                    iconFa: 'fa fa-star',
                  });
              }

              serviceDef.filters = [...commonFilters]
                .concat([
                  {
                    component: SelectFilter,
                    props: {
                      id: 'songType',
                      iconFa: 'fa fa-cubes',
                      title: 'Filter by map type',
                      values: [
                        {id: null, name: 'All'},
                        {id: 'ranked', name: 'Ranked only'},
                        {id: 'unranked', name: 'Unranked only'},
                      ],
                    },
                  },
                ]);
            }
            break;

          case 'beatsavior':
            serviceDef.filters = [...commonFilters];
            break;

          case 'accsaber':
            serviceDef.filters = [...commonFilters];

            if (accSaberCategories?.length) {
              const typeComponent = serviceDef.switcherComponents.find(c => c?.key === 'type');
              if (typeComponent)
                typeComponent.props = {
                  values: accSaberCategories.map(c => ({
                    id: c.name,
                    'label': c.displayName ?? c.name,
                    url: `/u/${playerId}/${service}/${c.name}/recent/1`,
                  })),
                }
            }
            break;
        }

        serviceDef.switcherComponents = serviceDef.switcherComponents
          .filter(c => c?.props)
          .map(c => {
            const key = c?.key ?? 'sort';

            [
              {propKey: 'value', compareObj: serviceParams},
              {propKey: 'loadingValue', compareObj: loadingServiceParams},
            ].forEach(o => c.props[o.propKey] = c.props?.values?.find(v => v?.id === o.compareObj?.[key]) ?? null)

            return c;
          })

        if (!serviceDef?.switcherComponents?.length) return null;

        return serviceDef;
      })
    .filter(s => s)
  }

  function onServiceChanged(event) {
    if (!event?.detail?.id) return;

    dispatch('service-change', event.detail.id)
  }

  function onFiltersChanged(event) {
    const newFilters = event?.detail ?? {}

    const {sort, order, ...filters} = newFilters;

    const changesToPush = {
      ...(sort? {sort} : null),
      ...(order? {order} : null),
      ...(filters ? {filters} : {filters: {}})
    }

    dispatch('service-params-change', changesToPush)
  }

  $: updateAvailableServiceNames(playerId)
  $: availableServices = updateAvailableServices(availableServiceNames, service, loadingService, serviceParams, loadingServiceParams, accSaberCategories)

  $: serviceObj = availableServices.find(s => s.id === service);
  $: loadingServiceObj = availableServices.find(s => s.id === loadingService)
</script>

<nav>
  <Switcher values={availableServices} value={serviceObj} on:change={onServiceChanged}
            loadingValue={loadingServiceObj}/>

  {#if serviceObj?.switcherComponents?.length}
    {#each serviceObj.switcherComponents as component (`${serviceObj?.id ?? ''}${component.key ?? 'sort'}`)}
      <svelte:component this={component.component} {...component.props}
                        on:change={component.onChange ?? null}
      />
    {/each}
  {/if}

  {#if serviceObj?.filters}
    {#key `${playerId}${service}`}
      <ScoreServiceFilters filters={serviceObj.filters} on:change={onFiltersChanged}/>
    {/key}
  {/if}
</nav>

<style>
    nav {
        display: flex;
        justify-content: space-evenly;
        align-items: flex-start;
        flex-wrap: wrap;
    }

    nav :global(> *) {
        margin-bottom: 1rem;
        margin-right: .75rem;
    }

    nav :global(> *:last-child) {
        margin-right: 0;
    }
</style>