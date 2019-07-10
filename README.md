# ATK Workflow
Implement workflow management in ATK and Saasty using Actions

## Design Goal
ATK actions can already be implemented using a PHP code such as Trait, Method etc. 

In some cases, however, it's important to define those actions in a more flexible form, through a set of rules. The pattern allows us to implement a workflow and state diagrams in models.

This add-on implements the following:

 - [ ] Handler for "addAction" which does not require PHP code, instead will read configuration array and will modify state.
 - [ ] State expressed through single, multiple fields in current or related model (e.g. containsMany)
 - [ ] Admin UI allowing to describe states and transitions, storing config
 
## Admin UI 
 
 TBC
 
## Configuration Format
 
Configuration is stored as an array. Here is the representation using yaml format for approving "Loan Application" model records.
 
``` yaml
States:
  - name: "Waiting for Review"
    conditions:
     - status: review
    actions:
     - name: Approve
       do:
        - set: { status: confirmed }
        - executeAfter: { sendEmail: [ 'approved-tpl', {field: contactEmail }  ] }

  - name: "Submit for Review"
    conditions:
     - status: draft
     - data: { condition: 'not null' } 
    actions:
     - do:
        - set: { status: review }

```
 
For ATK integration, you can place this inside your Model:

``` php
class LoanApplication extends \atk4\data\Model {
  use \atk4\workflow\WorkflowTrait;

  protected $workflow_config = ..yaml from above..;

  function init() {
    parent::init();
    
    // your init here......
    $this->addField('status', ['enum'=>['draft','review','confirmed']]);
    
    $this->initWorkflow($this->workflow_config);
  }
}
```

Obviously you can load config from database or a file, whichever is comfortable for you. After the code is executed the following will happen:

 - Two new actions will be created
 - afterLoad hook added to enable/disable actions depending on the `status` field value
 - valid callback for the actions that will transition to defined state
 
None of that requries ATK UI, so can be used in APIs or vanilla ATK Data
 
## Admin UI (optional)
 
Admin UI is implemented using a regular ATK UI, so you can easily add it to any page:
 
``` php
$admin = $page->add(\atk4\workflow\View\Admin::class);

// either specify model and a field where to store config:
$admin->setModel($m, ['config']);

// or provide raw config data and a callback for saving updated data
//  $admin->setSource($config_var, function($new_config) { ... save it here .. });
