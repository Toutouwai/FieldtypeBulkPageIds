# Fieldtype Bulk Page IDs

_Note: I created this module because I needed the functionality on a particular project. The constraints of the inputfield may limit its usefulness to others._

A fieldtype that stores and returns an array of page IDs, useful if you need to store a large amount of pages and you don't want to load a huge PageArray as the field value.

The fieldtype extends FieldtypePage so you can match by subfields of the stored pages. Example for field named "bulk_page_ids":

```php
$results = $pages->find("bulk_page_ids.title%=foo");
```

Unlike a Page Reference field there are no limitations about what kind of pages (template, parent, etc) may be stored in the field.

## Inputfield

The bundled inputfield module shows an alphabetical listing of the titles of the stored pages (optionally linked to Page Edit) but does not allow the field value to be changed. Changes to the field value must be done by setting an array of page IDs via the API.

![bulk-page-ids](https://user-images.githubusercontent.com/1538852/119912938-99c66f00-bfb0-11eb-92a5-6f9d61de8d2c.gif)

On the NZPCN website I allowed basic editing by adding single PageAutocomplete fields to the template: "Add a species" and "Remove a species". This hook updates the Bulk Page IDs field on saveReady:

```php
$pages->addHookAfter('saveReady', function(HookEvent $event) {
    /* @var Page $page */
    $page = $event->arguments(0);
    // Plant list (tabular)
    if($page->template == 'plant_list') {
        // Add a species ID
        if($page->flora_species->id) {
            $species = $page->species_ids;
            if(!in_array($page->flora_species->id, $page->species_ids)) {
                $species[] = $page->flora_species->id;
                $page->species_ids = $species;
            }
            $page->flora_species = '';
        }
        // Remove a species ID
        if($page->flora_species_2->id) {
            $species = $page->species_ids;
            if(($key = array_search($page->flora_species_2->id, $species)) !== false) {
                unset($species[$key]);
                $page->species_ids = $species;
            }
            $page->flora_species_2 = '';
        }
    }
});
```