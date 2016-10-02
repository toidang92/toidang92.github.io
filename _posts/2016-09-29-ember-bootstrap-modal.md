---
layout: post
title: Ember - Bootstrap Modal
---

## Getting started

The bellow will be used on our demo:

* Ember 2.6.1
* JQuery 2.1.4
* Emblem 0.9 (This is a template for ember. You can use Handlebars instead of this).
* ember-route-action-helper 2.0.0
* ember-wormhole 0.4.1 (There are a issue about display modal-backdrop so I have used it to resolve).

## Let's start

### 1\. Add modal outlet to our application template

{% highlight fade %}
{% raw %}
#ember-modal-container
= outlet 'modal'
{% endraw %}
{% endhighlight %}

* The `#ember-modal-container` div is where `ember-wormhole` will be moved block code inside.
* `outlet 'modal'` is the place where the template of modal component will be rendered.

### 2\. Add `openModal` and `closeModal` actions to application route

{% highlight javascript %}
{% raw %}
const MODAL_DEFAULT_DESTINATION = 'ember-modal-container';
const MODAL_DEFAULT_OUTLET = 'modal';

const { run, on } = Ember;

export default Ember.Route.extend({
  currentLayout: 'application',

  actions: {
    openModal(modalName, model, modalDefaultDestination = MODAL_DEFAULT_DESTINATION) {
      this.render(`modals/${modalName}`, {
        model,
        outlet: MODAL_DEFAULT_OUTLET,
        into: this.get('currentLayout'),
        emberWormholeDestination: modalDefaultDestination,
      });
    },

    closeModal() {
      this.disconnectOutlet({
        outlet: MODAL_DEFAULT_OUTLET,
        parentView: this.get('currentLayout')
      });
    }
  }
});
{% endraw %}
{% endhighlight %}

* openModal:
	* `modalName`: name of controller will be rendered in to modal.
	* `model`: the model of controller
	* `modalDefaultDestination`: is where `ember-wormhole` will be moved block code inside. Here is `ember-modal-container` div.


### 3\. Add template/component for modal

Here is my 2 components for modal:

#### 1\. bootstrap/bs-modal

* Component

{% highlight javascript %}
{% raw %}
const { computed, observer } = Ember;
const Modal = {};

Modal.TRANSITION_DURATION = 300;
Modal.BACKDROP_TRANSITION_DURATION = 150;

const observeOpen = function() {
  if (this.get('open')) {
    this.show();
  } else {
    this.hide();
  }
};

export default Ember.Component.extend({
  emberWormholeDestination: 'ember-modal-container',
  extraClass: null,
  open: true,
  title: null,
  closeButton: true,
  fade: true,
  'in': false,
  backdrop: true,
  showBackdrop: false,
  keyboard: true,
  autoClose: true,

  modalId: computed('elementId', function() {
    return `${this.get('elementId')}-modal`;
  }),

  modalElement: computed('modalId', function() {
    return Ember.$(`#${this.get('modalId')}`);
  }).volatile(),

  backdropId: computed('elementId', function() {
    return `${this.get('elementId')}-backdrop`;
  }),

  backdropElement: computed('backdropId', function() {
    return Ember.$(`#${this.get('backdropId')}`);
  }).volatile(),

  usesTransition: computed('fade', function() {
    return Ember.$.support.transition && this.get('fade');
  }),

  size: null,
  backdropClose: true,
  renderInPlace: false,
  submitAction: null,
  closeAction: null,
  closedAction: null,
  openAction: null,
  openedAction: null,

  actions: {
    close() {
      if (this.get('autoClose')) {
        this.set('open', false);
      }
      this.sendAction('closeAction');
    },

    submit() {
      let form = this.get('modalElement').find('.modal-body form');
      if (form.length > 0) {
        // trigger submit event on body form
        form.trigger('submit');
      } else {
        // if we have no form, we send a submit action
        this.sendAction('submitAction');
      }
    }
  },

  _observeOpen: observer('open', observeOpen),

  takeFocus() {
    let focusElement = this.get('modalElement').find('[autofocus]').first();
    if (focusElement.length === 0) {
      focusElement = this.get('modalElement');
    }
    if (focusElement.length > 0) {
      focusElement.focus();
    }
  },

  show() {

    this.checkScrollbar();
    this.setScrollbar();

    Ember.$('body').addClass('modal-open');

    this.resize();

    let callback = function() {
      if (this.get('isDestroyed')) {
        return;
      }

      this.get('modalElement')
        .show()
        .scrollTop(0);

      this.handleUpdate();
      this.set('in', true);
      this.sendAction('openAction');

      if (this.get('usesTransition')) {
        this.get('modalElement')
          .one('bsTransitionEnd', Ember.run.bind(this, function() {
            this.takeFocus();
            this.sendAction('openedAction');
          }))
          .emulateTransitionEnd(Modal.TRANSITION_DURATION);
      } else {
        this.takeFocus();
        this.sendAction('openedAction');
      }
    };
    Ember.run.scheduleOnce('afterRender', this, this.handleBackdrop, callback);
  },

  hide() {
    this.resize();
    this.set('in', false);

    if (this.get('usesTransition')) {
      this.get('modalElement')
        .one('bsTransitionEnd', Ember.run.bind(this, this.hideModal))
        .emulateTransitionEnd(Modal.TRANSITION_DURATION);
    } else {
      this.hideModal();
    }
  },

  hideModal() {
    if (this.get('isDestroyed')) {
      return;
    }

    this.get('modalElement').hide();
    this.handleBackdrop(() => {
      Ember.$('body').removeClass('modal-open');
      this.resetAdjustments();
      this.resetScrollbar();
      this.sendAction('closedAction');
    });
  },

  handleBackdrop(callback) {
    let doAnimate = this.get('usesTransition');

    if (this.get('open') && this.get('backdrop')) {
      this.set('showBackdrop', true);

      if (!callback) {
        return;
      }

      let waitForFade = function() {
        let $backdrop = this.get('backdropElement');
        Ember.assert('Backdrop element should be in DOM', $backdrop && $backdrop.length > 0);

        if (doAnimate) {
          $backdrop
            .one('bsTransitionEnd', Ember.run.bind(this, callback))
            .emulateTransitionEnd(Modal.BACKDROP_TRANSITION_DURATION);
        } else {
          callback.call(this);
        }
      };

      Ember.run.scheduleOnce('afterRender', this, waitForFade);

    } else if (!this.get('open') && this.get('backdrop')) {
      let $backdrop = this.get('backdropElement');
      Ember.assert('Backdrop element should be in DOM', $backdrop && $backdrop.length > 0);

      let callbackRemove = function() {
        this.set('showBackdrop', false);
        if (callback) {
          callback.call(this);
        }
      };
      if (doAnimate) {
        $backdrop
          .one('bsTransitionEnd', Ember.run.bind(this, callbackRemove))
          .emulateTransitionEnd(Modal.BACKDROP_TRANSITION_DURATION);
      } else {
        callbackRemove.call(this);
      }
    } else if (callback) {
      callback.call(this);
    }
  },

  resize() {
    if (this.get('open')) {
      Ember.$(window).on('resize.bs.modal', Ember.run.bind(this, this.handleUpdate));
    } else {
      Ember.$(window).off('resize.bs.modal');
    }
  },

  handleUpdate() {
    this.adjustDialog();
  },

  adjustDialog() {
    let modalIsOverflowing = this.get('modalElement')[0].scrollHeight > document.documentElement.clientHeight;
    this.get('modalElement').css({
      paddingLeft: !this.bodyIsOverflowing && modalIsOverflowing ? this.get('scrollbarWidth') : '',
      paddingRight: this.bodyIsOverflowing && !modalIsOverflowing ? this.get('scrollbarWidth') : ''
    });
  },

  resetAdjustments() {
    this.get('modalElement').css({
      paddingLeft: '',
      paddingRight: ''
    });
  },

  checkScrollbar() {
    let fullWindowWidth = window.innerWidth;
    if (!fullWindowWidth) { // workaround for missing window.innerWidth in IE8
      let documentElementRect = document.documentElement.getBoundingClientRect();
      fullWindowWidth = documentElementRect.right - Math.abs(documentElementRect.left);
    }

    this.bodyIsOverflowing = document.body.clientWidth < fullWindowWidth;
  },

  setScrollbar() {
    let bodyPad = parseInt((Ember.$('body').css('padding-right') || 0), 10);
    this.originalBodyPad = document.body.style.paddingRight || '';
    if (this.bodyIsOverflowing) {
      Ember.$('body').css('padding-right', bodyPad + this.get('scrollbarWidth'));
    }
  },

  resetScrollbar() {
    Ember.$('body').css('padding-right', this.originalBodyPad);
  },

  scrollbarWidth: computed(function() {
    let scrollDiv = document.createElement('div');
    scrollDiv.className = 'modal-scrollbar-measure';
    this.get('modalElement').after(scrollDiv);
    let scrollbarWidth = scrollDiv.offsetWidth - scrollDiv.clientWidth;
    Ember.$(scrollDiv).remove();
    return scrollbarWidth;
  }),

  didInsertElement() {
    if (this.get('open')) {
      this.show();
    }
  },

  willDestroyElement() {
    Ember.$(window).off('resize.bs.modal');
    Ember.$('body').removeClass('modal-open');
  }
});
{% endraw %}
{% endhighlight %}

* Template

{% highlight javascript %}
{% raw %}
= ember-wormhole to=emberWormholeDestination renderInPlace=renderInPlace
  = bootstrap/bs-modal-dialog close=(action "close") fade=fade in=in id=modalId title=title closeButton=closeButton keyboard=keyboard size=size extraClass=extraClass backdropClose=backdropClose
    yield this
  if showBackdrop
    .modal-backdrop class="modal-backdrop {{if fade "fade"}} {{if in "in"}}" id="{{backdropId}}"
{% endraw %}
{% endhighlight %}

#### 2\. bootstrap/bs-modal-dialog

* Component

{% highlight javascript %}
{% raw %}
import Ember from 'ember';

const { computed } = Ember;

export default Ember.Component.extend({
  classNames: ['modal', 'modal-center'],
  classNameBindings: ['fade', 'in'],
  attributeBindings: ['tabindex'],
  ariaRole: 'dialog',
  tabindex: '-1',

  title: null,
  closeButton: true,
  fade: true,
  'in': false,
  keyboard: true,
  size: null,
  backdropClose: true,

  sizeClass: computed('size', function() {
    let size = this.get('size');
    return Ember.isBlank(size) ? null : `modal-${size}`;
  }),

  keyDown(e) {
    let code = e.keyCode || e.which;
    if (code === 27 && this.get('keyboard')) {
      this.sendAction('close');
    }
  },

  click(e) {
    if (e.target !== e.currentTarget || !this.get('backdropClose')) {
      return;
    }
    this.sendAction('close');
  }
});
{% endraw %}
{% endhighlight %}

* Template

{% highlight javascript %}
{% raw %}
.modal-dialog class="{{sizeClass}} {{extraClass}}"
  .modal-content
    if title
      h4.modal-header.text-xs-center
        = title
    = yield
{% endraw %}
{% endhighlight %}

### 4\. How to use

Example:

{% highlight javascript %}
{% raw %}
i.fa.fa-share-square-o.card-share click="action 'openModal' 'quote/sharing' quote"
{% endraw %}
{% endhighlight %}

* `quote/sharing`: the controller which you want to display on modal;
* `quote`: the model of this controller.

## Bounty

If I want to modify the URL without reloading the page when the modal is opened that like facebook :D. You can use bellow code:

{% highlight javascript %}
{% raw %}
const MODAL_DEFAULT_DESTINATION = 'ember-modal-container';
const MODAL_DEFAULT_OUTLET = 'modal';

const { run, on } = Ember;

export default Ember.Route.extend({
  currentLayout: 'application',
  beforeModalUrl: null,

  actions: {

    openModal(modalName, model, replaceCurrentUrl = true, modalDefaultDestination = MODAL_DEFAULT_DESTINATION) {
      Ember.run.next(() => {
        this.set('beforeModalUrl', this.get('router.url'));

        if (replaceCurrentUrl) {
          let url = null;
          let modalRouteName = `${model.constructor.modelName}.index`;
          let urlOpts = { display: 'popup' };
          url = `${this.router.generate(modalRouteName, model)}?${Ember.$.param(urlOpts)}`;

          if (url) {
            window.history.replaceState({}, modalRouteName, url);
          }
        }

        this.render(`modals/${modalName}`, {
          model,
          outlet: MODAL_DEFAULT_OUTLET,
          into: this.get('currentLayout'),
          emberWormholeDestination: modalDefaultDestination,
        });
      });
    },

    closeModal() {
      this.disconnectOutlet({
        outlet: MODAL_DEFAULT_OUTLET,
        parentView: this.get('currentLayout')
      });

      if (this.get('beforeModalUrl')) {
        Ember.run(() => {
          window.location.href = this.router.location.formatURL(this.get('beforeModalUrl'));
          this.set('beforeModalUrl', null);
        });
      }
    }
  }
});
{% endraw %}
{% endhighlight %}

* `replaceCurrentUrl`: enable/disable modify the URL
* `beforeModalUrl`: store preview url
