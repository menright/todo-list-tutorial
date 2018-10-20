# Adding a checkbox

We are now able to interact with our todo list by removing items. But what if we want to complete items and still be able to see them in our list, with a line through the item's title? Enter the checkbox!

We will look at:

* Adding a checkbox
* Adding functionality when you click the checkbox so that a CSS class, which adds a ~~strikethrough~~ style, is added to our todo items
* Editing the todo title so that it responds to the checkbox
* Adding a new CSS Class

Let's go ahead and add a checkbox into our `todo-item.component.ts` file. Place the following code right before `{{ item.title }}`:

{% code-tabs %}
{% code-tabs-item title="src/app/todo-item/todo-item.component.ts" %}
```markup
<input type="checkbox"/>
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Now, in order for the checkbox to do anything, we need to add a `click` event handler which we will call `completeItem`. We'll also add a css-class and wrap the element and the interpolation together for styling. Let's do that now:

{% code-tabs %}
{% code-tabs-item title="src/app/todo-item/todo-item.component.ts" %}
```markup
<div>
  <input type="checkbox"
         class="todo-checkbox"
         (click)="completeItem()"/>
  {{ item.title }}
</div>
```
{% endcode-tabs-item %}
{% endcode-tabs %}

When we click on the checkbox, it will run the `completeItem` method. Let's talk about what this method needs to accomplish. We want to be able to toggle some CSS styling on the item's title so that when the checkbox is checked it will have a  strikethrough. We also want to save the status of the item in the local storage. In order to achieve this, we will emit an update event with the new status of the item and catch it in the parent component.

```javascript
export class TodoItemComponent implements OnInit {
  @Input() item: TodoItem;
  @Output() remove: EventEmitter<TodoItem> = new EventEmitter();
  @Output() update: EventEmitter<any> = new EventEmitter();

  // put this method below ngOnInit
  completeItem() {
    this.update.emit({
      item: this.item,
      changes: {completed: !this.item.completed}
    });
  }
```

But wait! How is any of this going to affect the todo title when we're only touching the checkbox? Well, Angular has this wonderful directive called NgClass. This directive applies or removes a CSS class based on a boolean \(true or false\) expression. There are many ways to use this directive \(see the [NgClass directive documentation](https://angular.io/api/common/NgClass)\) but we will focus on using it like so:

```markup
<some-element [ngClass]="{'first': true, 'second': true, 'third': false}">...</some-element>
```

The 'first' and 'second' class will be applied to the element because they are given a true value, whereas the 'third' class will not be applied because it is given a false value. So this is where our earlier code comes into play. Our `completeItem` method will toggle between true and false values, thus dictating whether a class should be applied or removed.

Let's wrap the item title in a `<span>`, then use NgClass to apply the styling:

```markup
<span class="todo-title" [ngClass]="{'todo-complete': completed}">
  {{ item.title }}
</span>
```

We now need to handle the emitted event.  The list-manager currently handles 'remove' events, but doesn't yet handle 'update' events.  We can add the following to list-manager.component.html to handle the update event, which contains an item and changes:

```markup
 <todo-item [todoItem]="item"  
      (update)="updateItem($event.item, $event.changes)" 
      (remove)="removeItem($event)"></todo-item>
```

We should now add a function to the component that calls the service with the event parameters in our list-manager.component.ts file:
```markup
  updateItem(item, changes) {
    this.todoList = this.todoListService.updateItem(item, changes);
  }
```

And finally, add the CSS to our `item.component.css` file:

```css
  .todo-complete {
    text-decoration: line-through;
  }
```

Voila! Checking the checkbox should apply a line through the todo title, and unchecking the checkbox should remove the line.

