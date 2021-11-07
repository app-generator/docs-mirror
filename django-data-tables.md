This page explains how to paginate data in Django using data tables. 

## Pagination in Django Views

Django's pagination helps you handle data divided across several pages. 
Django provides a **Paginator** class which takes a **queryset** and split this **queryset** into Page objects. These objects are then displayed across these pages while still using the same HTML document.

## Write paginator

We'll be writing a custom paginator here. As we want to create a partial custom template for this paginator. 

```python

import math
from urllib.parse import urlencode

from django.core.paginator import Paginator
from django.template.loader import render_to_string


def set_pagination(request, items, items_number=10):
    if not items:
        return True, "These is no items"

    params = request.GET
    item_len = len(items)
    page = int(params.get("page")) if "page" in params else 1
    pages_number = math.ceil(item_len / items_number)

    if page > pages_number or page <= 0:
        return False, "Page not found"

    paginator = Paginator(items, items_number)
    items = paginator.get_page(page)

    url_params = dict()
    for key in params:
        if key != 'page':
            url_params[key] = params[key]

    page_range = None
    if page in range(1, 7) and pages_number >= 7:
        page_range = [i for i in range(1, 8)]
        page_range += ['...']
    elif page >= 7 and (page + 6) < pages_number:
        page_range = ['...']
        page_range += [i for i in range(page - 3, page + 4)]
        page_range += ['...']
    elif page in range(pages_number - 7, pages_number + 1):
        page_range = ['...']
        page_range += [i for i in range(pages_number - 7, pages_number + 1)]

    context = dict(items=items, page_range=page_range, last=pages_number, url_params=urlencode(url_params))
    items.pagination = render_to_string('transactions/partial/pagination.html', context)
    return items, {'current_page': page, 'items': item_len, 'items_on_page': items_number}
```

The function is ready. As you can see, we are splitting the `queryset` according to `items_number` argument, which is equal to 10 by default. Let's write the template for this.

## Create pagination partial template

Writing a template for the paginator will make it easily reusable.

```html
{% if items.has_previous or items.has_next %}
    <div class="card-footer px-3 border-0 d-flex align-items-center justify-content-between">
        <nav aria-label="Page navigation example">
            <ul class="pagination mb-0">
                <li class="page-item">
                    <a class="page-link" {% if items.has_previous %}href="?page={{ items.previous_page_number }}{% if url_params %}&{{ url_params }}{% endif %}"{% endif %}>Previous</a>
                </li>

                {% for page_number in page_range %}
                    {% if items.number == page_number %}
                        <li class="page-item active">
                            <a class="page-link">{{ page_number }}</a>
                        </li>
                    {% else %}
                        <li class="page-item">
                            <a class="page-link" {% if page_number != '...' %}href="?page={{ page_number }}{% if url_params %}&{{ url_params }}{% endif %}"{% endif %}>{{ page_number }}</a>
                        </li>
                    {% endif %}
                {% endfor %}

                <li class="page-item">
                    <a class="page-link" {% if items.has_next %}href="?page={{ items.next_page_number }}{% if url_params %}&{{ url_params }}{% endif %}"{% endif %}>Next</a>
                </li>
            </ul>
        </nav>
        <div class="font-weight-bold small">Showing <b>{{ items.number }}</b> out of <b>{{ last }}</b> entries</div>
    </div>
{% endif %}
```

That's it. It'll make the paginator look like this.


![Paginator template](https://cdn.hashnode.com/res/hashnode/image/upload/v1636321828185/1IC0m57iq.png)

## Integrate it to a page

First of all, let's add it to a view. 

For example, the  [class-based view](https://github.com/app-generator/boilerplate-code-django-dashboard/blob/8e33315655b21b0cf35939e3c6c6bc2a7861ba6b/apps/datatables/views.py#L74)  for the transactions form of this project,  let's add a line before making returning the list of transactions when there is `GET` request. 

```python
...
        transactions = Transaction.objects.filter(filter_params) if filter_params else Transaction.objects.all()

        self.context['transactions'], self.context['info'] = set_pagination(request, transactions)
        if not self.context['transactions']:
            return False, self.context['info']

        return self.context, 'transactions/list.html'
...
```

And then let's add a line for the pagination of transactions objects in `transactions/list.html`.

```html
...
        {{ transactions.pagination }}
...
```
You can find an example of this code in the  [boilerplate-code-django-dashboard](https://github.com/app-generator/boilerplate-code-django-dashboard/blob/8e33315655b21b0cf35939e3c6c6bc2a7861ba6b/apps/templates/transactions/list.html#L116)

And we are done. 

You can find a functional code for this feature  [here](https://github.com/app-generator/boilerplate-code-django-dashboard/tree/master/apps/datatables).  

