- Start Date: 2021-07-15
- RFC PR: (leave this empty)
- Blaze Issue (JIRA): https://byte-9.atlassian.net/browse/BZ2-2583

# Summary

We need to be able to flexibly theme components so that they can be used in multiple scenarios in a developer friendly way. 

# Motivation

The core motivations for adding theming support are
- Extend tailwind usage within the component library
- Facilitate the improvement of the Blaze admin and frontend apps to make them more customisable
- Make a developer friendly component library

In addition to these there are some potential Blaze admin changes coming up that aim to show the frontend representation of pages in the admin when creating them in page builder and allow different elements of the page to have their styles customised. This will mean that Blaze components maybe used in the following scenarios and we will need a theming/styling system that is able to enable these

## Scenarios
### General
- components that require different sets of classes/styles e.g these components might require classes for
  - input
    - wrapper
    - label
    - input
    - validation message
  - multiselect
    - wrapper
    - label
    - search input
    - checkboxes
    - chips
    - ...
- different variations of a component e.g. accept/confirm buttons
  - this might be done by applying different classes
  - might not be a concept we need
### Admin 
- admin standard use in byte9 plugin e.g. login form
- third party plugin created by outside dev. e.g. add new admin page with a form
- rendering components inside page builder frontend editing/view
  - frontend representation of button component
  - standard button variants used inside other components e.g. search filters or forms
- applying/collection design overrides for components and sub components inside page builder frontend editing/view
  - search filter button would pick up default button styles for FE. But these could be overriden in page builder admin/settings
  - should just save/track changed design options so that if the defaults change the component handles e.g
    - default FE button is square and orange
    - you create a button and change the colour from default to green
    - FE theme changes defaults buttons to be rounded
    - your custom button also becomes rounded but stays green  
- component inside another component e.g button inside modal footer
### Frontend
- rendering a component defined in page builder e.g. button component
- components inside other components e.g. search filter submit button
- third party plugin/component created by outside dev. e.g. add new component with a button/input

# Detailed design

This is currently under proof of concept testing but general principles are
- Create stucture for a theme config that can define classes for
    - components e.g. button
    - variants e.g. secondary button
- Add a theme provider that accepts the theme config
- Components should indirectly get their default styling from the theme config set in the provider
- It should be possible to pass override styles/classes to a component to override the defaults

Initial examples

```
const PRIMARY_BUTTON_CLASSES = {
  wrapper: 'classes...'
};

const SECONDARY_BUTTON_CLASSES = {
  wrapper: 'classes...'
};

const MODAL_CLASSES = {
  wrapper: '',
  closeButton: PRIMARY_BUTTON_CLASSES,
  acceptButton: SECONDARY_BUTTON_CLASSES
}


const theme = {
    button: {
        defaults: {
            wrapper: 'bg-default',
        }
        variants: {
            primiary: {
                wrapper: 'bg-primary',
            }
            secondary:  {
                wrapper: 'bg-secondary',
            }
        }
    },
    modal: {
        defaults: {
            wrapper: '',
            closeButton: PRIMARY_BUTTON_CLASSES,
            acceptButton: SECONDARY_BUTTON_CLASSES
        }
    }
}

```

```js

// controller 
const ButtonController = ({ disabled, overrideClasses, children }) => {
 const theme = useContent(ThemeContext);

const classes = theme.button.variants.primary;
// util to merege with overrides? overrideClasses
if(disabled) classes.push('button-disabled');

return <Button utititis={} >
{children}
</Button>

// implement overrides
const ModalController = ({ utilities }) = > {
  const theme = useContent(ThemeContext);
  const noButtonClasses = utilities.components.noButton || theme.button.variants.secondary;
  const yesButtonClasses = utilities.components.yesButton || theme.button.variants.primary;
  return <div wrapper={utilities.wrapper}>
   <div id="body"> Modal text</div>
   <Button utitlities={ wrapper: noButtonClasses }>No</Button>
   <Button utitlities={ wrapper: yesButtonClasses }>Yes</Button>
  </div>
}

<BlazeInput variant="variant_key" disabled=true />
```

# Drawbacks

Why should we *not* do this? Please consider:

- implementation cost, both in term of code size and complexity
- whether the proposed feature can be implemented in user space
- the impact on teaching people to use Blaze
- integration of this feature with other existing and planned features
- cost of migrating existing Blaze applications (is it a breaking change?)

There are tradeoffs to choosing any path. Attempt to identify them here.

# Alternatives

What other designs have been considered? What is the impact of not doing this?

# Adoption strategy

- Initially update admin and apply the them for a selection of components
- Roll out across the admin
- Add to the frontend
- Implement further page builder design option overrides in the admin and frontend

# How we teach this

How to use the new features will be visible/promoted via the [Blaze react components storybook pages](https://blaze-components-react.thisisblaze.com/?path=/story/blaze-react-button-variations--page)

# Unresolved questions

- theme structure
- component class structure 
- shared component utils for managing fetching classes/overrides
- theme optimisation for bundle size

# Design references (If applicable): 
