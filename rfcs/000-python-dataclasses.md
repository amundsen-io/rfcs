- Feature Name: python-dataclasses
- Start Date: 2020-11-10
- RFC PR: [amundsen-io/rfcs#12](https://github.com/amundsen-io/rfcs/pull/12) (after opening the RFC PR, update this with a link to it and update the file name)
- Amundsen Issue: [amundsen-io/amundsen#0000](https://github.com/amundsen-io/amundsen/issues/0000) (leave this empty for now)

# Leverage Python's (native) Dataclasses instead of attrs (a third party package)

## Summary

- Python 3.7 introduced [dataclasses (PEP557)](https://docs.python.org/3/library/dataclasses.html) .Dataclasses can be a convenient way to generate classes whose primary goal is to contain values. 
- The design of dataclasses is based on the pre-existing attr.s library, and are basically a slimmed-down version of attrs. 

- In fact `Hynek Schlawack`, the very same author of attrs, helped with the writing of PEP557.

## Motivation

We are relying on `attrs` for (de-)serializing data for each request within Amundsen. 
`attrs` itself is a big package with so many different features available that we don't even need. 
By using Python's native dataclasses we can have one less dependency with almost the same features `attrs` provide that we intend to use.

## Guide-level Explanation (aka Product Details)

-  The details about the dataclasses can be found here: https://docs.python.org/3/library/dataclasses.html
-  Example of a dataclass:
```python
from dataclasses import dataclass, asdict

@dataclass
class InventoryItem:
    """Class for keeping track of an item in inventory."""
    name: str
    unit_price: float
    quantity_on_hand: int = 0

    def total_cost(self) -> float:
        return self.unit_price * self.quantity_on_hand

inventory = InventoryItem(name='alpha', unit_price=20.0, quantity_on_hand=5)

# OR directly from a dict
data = {"name": 'alpha', "unit_price": 20.0, "quantity_on_hand": 5}
inventory_alt = InventoryItem(**data)

# Can access fields directly by dot notations
assert inventory_alt == "alpha"

# and then convert it back to dict 
asdict(inventory_alt)
```

## UI/UX-level Explanation

Nothing changes on the UI/UX.

## Reference-level Explanation (aka Technical Details)

- The primary change will happen in amundsencommon repositories where we are defining the models. I believe no other major change 
will be required in any other repository. 

- Python 3.6 can still be supported as long as it's needed through  backported to py36 ([ericvsmith/dataclasses](https://github.com/ericvsmith/dataclasses)). 

## Drawbacks

- One of the main thing to consider is the Python 3.6 support. If we go for the dataclasses, we will not be supporting 
Python 3.6 anymore. The minimum version requirement for Python for Amundsen will become 3.7.

- `attrs` supports validation which is not there in Python dataclasses by design. Even though we are not using validations 
a lot (in fact, there is only one place in User model where we are using attrs' validation), but still we need to keep 
this in mind while making this change. This can be easily replicated within the `__init__` method of a dataclass if needed.
  

## Alternatives
Another package that supports the validation on Python's dataclasses is [Pydantic](https://pydantic-docs.helpmanual.io/). 
Again, I personally believe we are not doing any kind of validations in the serializer/deserializers, so not to make things 
complex, instead look for a simpler and cleaner solution. 


## Prior art
N/A

## Unresolved questions

- Should we aim for having as less dependencies as possible? That includes the third party packages. 
- Do we think of a case where we'll be doing heavy data validations, and if those can not be done with dataclasses?

## Future possibilities

- We'll be leveraging the native python features, and hence will be learning one new feature i.e., dataclass.
