# Steps to add field in admin form Drupal 8

Creating an admin form is often used. An admin interface enables you to make a module's settings configurable by editor or administrator
so new fields can be added or existing fields can be changed.
Here we are going to add new field called 'siteapikey'


Module Set Up 

+ Create a folder in /modules called custom
+ Inside create folder for your custom Module lets called as add_siteapi_key 
+ Create necessary files add_siteapi_key.info.yml, add_siteapi_key.services.yml, add_siteapi_key.install
+ Add Form in add_siteapi_key\src\Form\AddFieldToSiteInformationForm
+ Add RouteSubscriber in add_siteapi_key\src\Routing\RouteSubscriber


Code 

add_siteapi_key.info.yml

```
name: 'Add Site API Key to Site Configuration'
type: module
description: 'Add new field called site api key to site configurtaion form'
core: 8.x
package: 'Custom'

```

add_siteapi_key.install

```
function add_siteapi_key_uninstall() {
  // Remove set Site API Key configuration
  \Drupal::configFactory()->getEditable('system.site')->clear('siteapikey')->save();
}

```

add_siteapi_key.services.yml

```
services:
  add_siteapi_key.route_subscriber:
    class: \Drupal\add_siteapi_key\Routing\RouteSubscriber
    tags:
      - { name: event_subscriber }
```


AddFieldToSiteInformationForm.php

```
namespace Drupal\add_siteapi_key\Form;

use Drupal\Core\Form\FormStateInterface;
use Drupal\system\Form\SiteInformationForm;

/**
 * Add the SITE API key.
 */
class AddFieldToSiteInformationForm extends SiteInformationForm {

  /**
   * Create Field SITE API key.
   *
   * @param array $form
   *   An associative array containing the structure of the form.
   * @param \Drupal\Core\Form\FormStateInterface $form_state
   *   The current state of the form.
   */
  public function buildForm(array $form, FormStateInterface $form_state) {
    $site_config = $this->config('system.site');
    $form =  parent::buildForm($form, $form_state);
	$form['site_information']['siteapikey'] = [
      '#type' => 'textfield',
      '#title' => t('Site API Key'),
      '#default_value' => $site_config->get('siteapikey') ?: 'No API Key yet',
	  '#description' => t("Set the Site API Key"),
    ];
    $form['actions']['submit']['#value'] = t('Update configuration');
    return $form;
  }

  /**
   * {@inheritdoc}
   */  
  public function submitForm(array &$form, FormStateInterface $form_state) {
    $this->config('system.site')
      ->set('siteapikey', $form_state->getValue('siteapikey'))
      ->save();
    parent::submitForm($form, $form_state);

	// Add message that Site API Key has been set
    drupal_set_message("Successfully saved Site API Key - " . $form_state->getValue('siteapikey'));
  }

}

```


RouteSubscriber.php

```
namespace Drupal\add_siteapi_key\Routing;

use Drupal\Core\Routing\RouteSubscriberBase;
use Symfony\Component\Routing\RouteCollection;

/**
 * Listens to the dynamic route events.
 */
class RouteSubscriber extends RouteSubscriberBase {

  /**
   * Add Form Field for site settings form
   *
   * @param \Symfony\Component\Routing\RouteCollection $collection
   *   RouteCollection object.
   */
  protected function alterRoutes(RouteCollection $collection) {
    if ($route = $collection->get('system.site_information_settings')) { 
      $route->setDefault('_form', '\Drupal\add_siteapi_key\Form\AddFieldToSiteInformationForm');
    }	  
  }

}

```
