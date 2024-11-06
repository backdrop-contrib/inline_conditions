# Inline Conditions

It is common for a rule to be generated based on an entity (discounts, shipping
rates, etc).

This module allows conditions to be defined on the entity add / edit form, and
those conditions are later mapped to rules conditions when the rule is
generated.

Inline Conditions are specially defined (hook_inline_condition_info()) and
consist of a configure callback (provides a user-facing form) and a build
callback (adds the actual condition to the rule). Integration consists of
creating a field of the "inline_conditions" type on the entity, and later
calling inline_conditions_build_rule() from the implementation of
hook_default_rules_configuration(). See inline_conditions.api.php for more
information.

## Installation

* Install this module using the [official Backdrop CMS instructions](https://backdropcms.org/guide/modules)
* Install Rules.

## Define an inline condition

To define a new inline condition, the code has to be put in two files:

* `[module].rules.inc` => rule conditions and build callbacks.
* `[module].inline_conditions.inc` => inline conditions, configure callbacks and
  the others hook exposed by inline condition module.

### [module].inline_conditions.inc

```php
/**
 * Implements hook_inline_conditions_info().
 */
function commerce_discount_role_inline_conditions_info() {
  $conditions = array();

  $conditions['order_owner_has_role'] = array(
    'label' => t('Role'),
    'entity type' => 'commerce_line_item',
    'callbacks' => array(
      'configure' => 'commerce_discount_role_order_owner_has_role_configure',
      'build' => 'commerce_discount_role_order_owner_has_role_build',
    ),
  );

  return $conditions;
}

/**
 * Configuration callback for order_owner_has_role.
 *
 * @param array $settings
 *   An array of rules condition settings.
 *
 * @return array;
 *   A form element array.
 */
function commerce_discount_role_order_owner_has_role_configure($settings) {
  $form = array();
  $default_value = '';

  if (!empty($settings)) {
    $default_value = $settings['role'] != '' ? $settings['role'] : '';
  }

  $form['role'] = array(
    '#type' => 'select',
    '#title' => t('Role'),
    '#title_display' => 'invisible',
    '#options' => user_roles(TRUE),
    '#default_value' => $default_value,
    '#required' => TRUE,
    '#suffix' => '<div class="condition-instructions">' . t('Discount is active if the order owner has the selected role.') . '</div>',
  );

  return $form;
}
```

### [module].rules.inc

```php
/**
 * Implements hook_rules_condition_info().
 *
 * Adds new rule conditions to commerce_line_item entity type.
 */
function commerce_discount_role_rules_condition_info() {
  return array(
    'order_owner_has_role' => array(
      'label' => t('Order owner has role'),
      'parameter' => array(
        'commerce_line_item' => array(
          'type' => 'commerce_line_item',
          'label' => t('Line Item'),
          'description' => t('A product line item.'),
          'wrapped' => TRUE,
        ),
        'role' => array(
          'type' => 'text',
          'label' => t('Role'),
          'description' => t('Role.'),
        ),
      ),
      'module' => 'commerce_discount_role',
      'group' => t('Commerce Order'),
      'callbacks' => array(
        'execute' => 'commerce_discount_role_order_owner_has_role_build',
      ),
    )
  );
}

/**
 * Build callback for order_owner_has_role.
 *
 * @param EntityBackdropWrapper $line_item_wrapper
 *   The wrapped entity given by the rule.
 * @param integer $role
 *   role id
 *
 * @return bool
 *   Returns true if condition is valid. false otherwise.
 */
function commerce_discount_role_order_owner_has_role_build(EntityBackdropWrapper $line_item_wrapper, $role) {
  if ($order = commerce_order_load($line_item_wrapper->order_id->value()) && $user = user_load($order->uid)) {
    return isset($user->roles[$role]);
  }

  return FALSE;
}
```

## Current Maintainers

* Seeking maintainers.

## Credit

Originally maintained on Backdrop by:

* <https://www.drupal.org/u/bojanz>
* <https://www.drupal.org/u/czigor>
* <https://www.drupal.org/u/jsacksick>
* <https://www.drupal.org/u/jkuma>
* <https://www.drupal.org/u/joelpittet>
* <https://www.drupal.org/u/pcambra>

## License

This project is GPL v2 software. See the LICENSE.txt file in this directory for
complete text.
