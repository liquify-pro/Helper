document.addEventListener('DOMContentLoaded', () => {
  const filterWrapper = document.querySelectorAll('[li-render="filter-wrapper"]');
  const recWrapper = document.querySelectorAll('[li-render="recommendations-wrapper"]')
  const defaultWrapper = document.querySelectorAll('[li-render="default-wrapper"]')
  const searchWrapper = document.querySelectorAll('[li-render="predictive-search-wrapper"]')
  let filterQuery = new URLSearchParams();

  const createSearchQuery = (wrapper) => {
    const rType = wrapper.getAttribute('li-render-rt');
    const rLimit = wrapper.getAttribute('li-render-rl');
    const rLimitScope = wrapper.getAttribute('li-render-rls');
    const rOptionsUnavailableProducts = wrapper.getAttribute('li-render-roup');
    const rOptionsFields = wrapper.getAttribute('li-render-rof');

    let searchQuery = new URLSearchParams();

    if (rType) {
      searchQuery.append('resources[type]', rType);
    }

    if (rLimit) {
      searchQuery.append('resources[limit]', rLimit);
    }

    if (rLimitScope) {
      searchQuery.append('resources[limit_scope]', rLimitScope);
    }

    if (rOptionsUnavailableProducts) {
      searchQuery.append(
        'resources[options][unavailable_products]', rOptionsUnavailableProducts);
    }

    if (rOptionsFields) {
      searchQuery.append('resources[options][fields]', rOptionsFields);
    }

    return searchQuery
  }

  const setFilterState = () => {
    const searchParams = new URLSearchParams(window.location.search)
    if (searchParams) {
      filterQuery = searchParams
    }

    document.querySelectorAll('[li-render="filter"]').forEach(element => {
      const param = element.getAttribute('li-render-param-name')
      const value = element.getAttribute('li-render-value')
      const customCheckbox = element.parentElement.querySelector('div')

      if (!param || !value) return

      if (filterQuery.has(param, value)) {
        element.setAttribute('checked', 'checked')
        if (customCheckbox) {
          customCheckbox.classList.add('w--redirected-checked')
        }
      } else {
        element.removeAttribute('checked')
        if (customCheckbox) {
          customCheckbox.classList.remove('w--redirected-checked')
        }
      }
    })

    if (filterQuery.has('filter.v.price.gte')) {
      document.querySelectorAll('[li-render="price-min"]').forEach(element => {
        element.value = filterQuery.get('filter.v.price.gte')
      })
    }

    if (filterQuery.has('filter.v.price.lte')) {
      document.querySelectorAll('[li-render="price-max"]').forEach(element => {
        element.value = filterQuery.get('filter.v.price.lte')
      })
    }
  }

  const debounce = (callback, wait) => {
    let timeoutId = null;
    return (...args) => {
      window.clearTimeout(timeoutId);
      timeoutId = window.setTimeout(() => {
        callback(...args);
      }, wait);
    };
  }


  const renderSection = async (fetchUrl, wrapper, target = null) => {
    console.log("Render section function")
    try {
      const response = await fetch(fetchUrl);

      if (!response.ok) {
        throw new Error(
          `Error fetching section, response status: ${response.status}`
        );
      }

      const newHtml = await response.text();

      const parser = new DOMParser();
      const newDocument = parser.parseFromString(newHtml, 'text/html');

      const newElements = target ? newDocument.querySelectorAll(target) : newDocument
        .querySelectorAll(wrapper);

      if (newElements.length > 0) {
        const currentElements = target ? document.querySelectorAll(target) : document
          .querySelectorAll(wrapper);

        if (currentElements.length === newElements.length) {
          currentElements.forEach((currentElement, index) => {
            currentElement.innerHTML = newElements[index].innerHTML;
          });

          initializeFilters();
          setFilterState();

          const sectionsRendered = new CustomEvent("liquify:sections-rendered", {
            bubbles: true,
            cancelable: false,
            composed: false
          });

          document.dispatchEvent(sectionsRendered);
        } else {
          console.warn('Mismatch in element count between current and new content.');
        }
      }
    } catch (error) {
      console.error(error.message);
    }
  };


  searchWrapper.forEach((wrapper) => {
    const input = wrapper.querySelector('[li-render="predictive-search-input"]');
    const sectionId = wrapper.getAttribute('li-render-section-id');

    if (!sectionId) {
      console.warn("Failed to render section because of missing section id")
      return
    }

    const searchQuery = createSearchQuery(wrapper)

    if (input) {
      input.addEventListener(
        'input',
        debounce(() => {
          const searchTerm = input.value.trim();
          let searchUrl = ''

          if (searchQuery.size === 0) {
            searchUrl = `/search/suggest?q=${searchTerm}&section_id=${sectionId}`
          } else {
            searchUrl = `/search/suggest?q=${searchTerm}&section_id=${sectionId}&${searchQuery}`
          }

          if (searchTerm) {
            renderSection(searchUrl, '[li-render="predictive-search-wrapper"]',
              '[li-render="predictive-search-target"]')
          };
          console.log(searchUrl)
        }, 300)
      );
    }
  });


  const initializeFilters = () => {
    filterWrapper.forEach((wrapper) => {
      const submitButton = wrapper.querySelector('[li-render="submit-button"]')
      const filterTarget = wrapper.querySelector('[li-render="filter-target"]')

      const getQueryParam = (param = '', value = '', render = false, uniqueValue = false) => {

        if (filterQuery.has(param) && uniqueValue === true) {
          filterQuery.set(param, value)
        } else if (filterQuery.has(param, value) && uniqueValue === false) {
          filterQuery.delete(param, value)
        } else {
          filterQuery.append(param, value)
        }

        const completeFilterQuery = '?' + filterQuery

        if (!submitButton || render === true) {
          history.replaceState(null, '', window.location.pathname + completeFilterQuery);

          if (filterTarget) {
            renderSection(completeFilterQuery, '[li-render="filter-wrapper"]', '[li-render="filter-target"]');
          } else {
            renderSection(completeFilterQuery, '[li-render="filter-wrapper"]');
          }

          return;
        }

        if (submitButton) {
          submitButton.addEventListener('click', (event) => {
            event.preventDefault();
            history.replaceState(null, '', window.location.pathname + completeFilterQuery);

            if (filterTarget) {
              renderSection(completeFilterQuery, '[li-render="filter-wrapper"]', '[li-render="filter-target"]');
            } else {
              renderSection(completeFilterQuery, '[li-render="filter-wrapper"]');
            }
          });
        }
      };

      wrapper
        .querySelectorAll('[li-render="filter"]')
        .forEach((element) => {
          element.addEventListener("change", () => {
            const param = element.getAttribute('li-render-param-name')
            const value = element.getAttribute('li-render-value')
            getQueryParam(param, value);
          });
        });


      wrapper
        .querySelectorAll('[li-render="sort-select"]')
        .forEach((element) => {
          element.addEventListener("change", (event) => {
            event.preventDefault();
            getQueryParam('sort_by', element.value, true, true);
          });
        });

      wrapper
        .querySelectorAll('[li-render="filter-remove"]')
        .forEach((element) => {
          element.addEventListener("click", (event) => {
            event.preventDefault();
            const removeFilter = element.getAttribute('li-render-value')
            renderSection(removeFilter, '[li-render="filter-wrapper"]');
            history.replaceState(null, '', removeFilter);
          });
        });


      wrapper.querySelectorAll('[li-render="filter-search"]').forEach(element => {
        const form = element.closest('form')
        const searchQuery = createSearchQuery(wrapper)
        let inputValue = element.value

        element.addEventListener('input',
          debounce(() => {
            inputValue = element.value

            if (window.location.pathname === '/search') {
              getQueryParam('q', inputValue, undefined, true);
            }

            console.log(inputValue)
          }, 300)
        )

        if (inputValue !== '' && window.location.pathname !== '/search' && form) {
          form.addEventListener('submit', (event) => {
            event.preventDefault()
            form.action = `/search?${searchQuery}&q=${inputValue}`
            form.submit()
          })
        }
      })

      wrapper
        .querySelectorAll('[li-render="clear-all"]')
        .forEach((element) => {
          element.addEventListener('click', (event) => {
            event.preventDefault();
            const value = element.getAttribute('li-render-value')

            if (!value) {
              console.warn(`Unable to clear all filters because of one missing attribute.
                Please add the attribute [li-render-value="{{ routes.search_url }}"] on search pages or [li-render-value="{{ collection.url }}"] on collection pages to the [li-render="clear-all"] button`)
              return
            }

            filterQuery = new URLSearchParams()
            history.replaceState(null, '', window.location.pathname);
            renderSection(value, '[li-render="filter-wrapper"]')
          });
        });

      wrapper.querySelectorAll('[li-render="price-min"]').forEach(element => {
        element.addEventListener('input', () => {
          getQueryParam('filter.v.price.gte', element.value, false)
        })
      })

      wrapper.querySelectorAll('[li-render="price-max"]').forEach(element => {
        element.addEventListener('input', () => {
          getQueryParam('filter.v.price.lte', element.value, false)
        })
      })
    })
  };


  recWrapper.forEach((wrapper) => {
    const path = wrapper.getAttribute('li-render-path')
    const sectionId = wrapper.getAttribute('li-render-section-id')
    const productId = wrapper.getAttribute('li-render-product-id')
    const limit = wrapper.getAttribute('li-render-limit') || 4;
    const intent = wrapper.getAttribute('li-render-intent') || "related";

    if (!sectionId) {
      console.warn(
        'Unable to render product recommendations because of missing section ID. Please add the attribute li-render-section-id="{{ section.id }}" to the li-render="recommendations-wrapper" element'
      )
      return
    }

    if (!productId) {
      console.warn(
        'Unable to render product recommendations because of missing product ID. Please add the attribute li-render-product-id="{{ product.id }}" to the li-render="recommendations-wrapper" element'
      )
      return
    }

    let recQuery = new URLSearchParams()
    recQuery.append('section_id', sectionId)
    recQuery.append('product_id', productId)
    recQuery.append('limit', limit)
    recQuery.append('intent', intent)

    const fetchUrl = `${path}?${recQuery}`;
    renderSection(fetchUrl, '[li-render="recommendations-wrapper"]');
    console.log(fetchUrl)
  });



  defaultWrapper.forEach(wrapper => {
    const sectionId = wrapper.getAttribute('li-render-section-id')
    const variantId = wrapper.getAttribute('li-render-variant-id')
    const trigger = wrapper.querySelector('[li-render="trigger"]')
    const target = wrapper.querySelector('[li-render="target"]')

    if (!sectionId) {
      console.warn("Failed to render section because of missing section id")
      return
    }

    if (!variantId) {
      console.warn("Failed to render section because of missing variant id")
      return
    }

    if (trigger.hasAttribute('input')) {
      trigger.addEventListener('change', () => {

      })
    }

    const fetchUrl = `${window.location.pathname}?section_id=${sectionId}&variant=${variantId}`

    renderSection(fetchUrl, '[li-render="default-wrapper"]', '[li-render="target"]')

  })

  setFilterState()
  initializeFilters()
});
