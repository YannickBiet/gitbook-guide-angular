# La boite à outils des "Reactive Forms"

## `FormControl`

La classe **`FormControl`** **est une indirection permettant de contrôler et d'accéder à l'état** des "controls" de la vue _\(e.g. `<input>`\)_.

{% code-tabs %}
{% code-tabs-item title="book-form.component.ts" %}
```typescript
export class BookFormComponent {

    bookTitleControl = new FormControl();
    bookDescriptionControl = new FormControl();

    submitBook() {
        console.log(this.bookTitleControl.value);
        console.log(this.bookDescriptionControl.value);
    }

}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% code-tabs %}
{% code-tabs-item title="book-form.component.html" %}
```markup
<input
    type="text"
    [formControl]="bookTitleControl">

<textarea [formControl]="bookDescriptionControl"></textarea>

<button
    type="submit"
    (click)="submitBook()">SUBMIT</button>
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% hint style="warning" %}
On obtient l'erreur suivante :

`Can't bind to 'formControl' since it isn't a known property of 'input'.`

... car l'`Input` `formControl` est ajouté via la "directive" `FormControlDirective` _\(_[_https://github.com/angular/angular/blob/master/packages/forms/src/directives/reactive\_directives/form\_group\_directive.ts_](https://github.com/angular/angular/blob/master/packages/forms/src/directives/reactive_directives/form_group_directive.ts)_\)_ mais cette "directive" n'est pas importée nativement.

Pour profiter de la directive `FormGroupDirective`, **il faut importer le module `ReactiveFormsModule`** dans les modules contenant des composants qui en dépendent.

```typescript
import { ReactiveFormsModule } from '@angular/forms';

@NgModule({
    ...
    imports: [
        ReactiveFormsModule
    ]
})
export class BookModule {
}
```

**Vous pouvez retirer l'import du module `FormsModule`.**
{% endhint %}

 `FormControl` fournit différentes propriétés et méthodes permettant de piloter le "control" :

* **`value`** : permet d'accéder à la valeur actuellement contenu dans le "control".
* **`valueChanges`** : est un `Observable` permettant d'**observer les changements** de valeur du "control".
* **`reset`**, **`setValue`** et **`patchValue`** permettent de modifier l'état et la valeur du "control".

{% hint style="danger" %}
La propriété `value` est une source dynamique de données et elle est donc de type `any`.

Méfiez-vous en lors de son utilisation car elle est donc compatible avec tous les types et vous ne serez donc pas "protégé" par la compilation TypeScript.
{% endhint %}

## `FormGroup`

`FormGroup` permet de **regrouper des "controls"** afin de faciliter l'implémentation, récupérer la valeur groupée des "controls" ou encore appliquer des validateurs au groupe.

Le `FormGroup` est construit avec un "plain object" associant chaque **propriété** à un **`FormControl`** .  
Il est alors possible d'accéder aux valeurs et aux "controls" via la propriété associée.

{% code-tabs %}
{% code-tabs-item title="book-form.component.ts" %}
```typescript
export class BookFormComponent {

    bookForm = new FormGroup({
        title: new FormControl(),
        description: new FormControl()
    });

    submitBook() {
        console.log(this.bookForm.value);
    }

}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% code-tabs %}
{% code-tabs-item title="book-form.component.html" %}
```markup
<input
        type="text"
        [formControl]="bookForm.controls.title">

<textarea [formControl]="bookForm.controls.description"></textarea>

<button
        type="submit"
        (click)="submitBook()">SUBMIT</button>
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% hint style="info" %}
L'objet `this.bookForm.value` présente l'ensemble des valeurs des controls via un "plain objet" :  
`{title: 'TITLE', description: null}`
{% endhint %}



{% hint style="success" %}
Pour éviter d'accéder aux controls via la propriété `FormGroup.controls`, les "Reactive Forms" implémentent également une directive `FormGroupDirective` qui permet d'**associer un `FormGroup` à un élément du DOM** _**\(`form` en général mais on peut utiliser d'autres éléments\)**_. Cela permet alors d'associer  les "controls" via leur nom à l'aide de l'`Input` `formControlName`. 

{% code-tabs %}
{% code-tabs-item title="book-form.component.html" %}
```markup
<form
        [formGroup]="bookForm"
        (ngSubmit)="submitBook()">

    <input
            type="text"
            formControlName="title">
            
    <textarea formControlName="description"></textarea>
    
    <button type="submit">SUBMIT</button>
    
</form>
```
{% endcode-tabs-item %}
{% endcode-tabs %}
{% endhint %}

La classe **`FormGroup`** propose de nombreuses **propriétés et méthodes similaires** à celles rencontrées précédemment avec **`FormControl`**. En effet, **`FormControl` et `FormGroup` héritent de la classe abstraite `AbstractControl`**.

Un `FormGroup` est donc également un "control" et il est donc possible de construire une **arborescence de `FormGroup`**s afin de **regrouper différentes parties d'un formulaire de taille importante** ou dont des parties sont **réutilisables dans d'autres contextes**.

```typescript
new FormGroup({
    address: new FormGroup({
        street: new FormControl()
    })
});
```

## `FormArray`

`FormArray` héritant également de la classe abstraite `AbstractControl`,  **permet de créer un "control" contenant une liste de "controls"** _\(e.g. `FormControl` ou `FormGroup`\)_ **ordonnés** _\(contrairement à `FormGroup` qui permet de créer un groupe de "controls" accessibles avec des clés sur un "plain object"\)._

La valeur contenu dans le `FormArray` est de type `Array`.

## `FormBuilder`

Le module `ReactiveFormsModule` implémente également un service `FormBuilder` qui permet d'ajouter un peu de "syntactic sugar" afin de **simplifier la création des `FormGroup`, `FormArray` ou `FormControl`**.

```typescript
bookForm: FormGroup;

constructor(private _formBuilder: FormBuilder) {
    this.bookForm = this._formBuilder.group({
        title: null,
        description: null
    });
}
```

 

