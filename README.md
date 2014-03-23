Parinx
======
Parinx implements a basic Sphinx docstring parser language which provides
a interface to extract the relavant parameter. You might find
it most useful for tasks involving automated data extraction from sphinx
docs. Typical usage
often looks like this::

    #!/usr/bin/env python

    from parinx import parser

    def test_parse_docstring(self):
        docstring = """
        Return a dict.

        :type zone_id: ``str``
        :param zone_id: Required zone id (required)
        :keyword    auth:   Initial authentication information for the node
                            (optional)

        :rtype:    ``dict``
        """
        result = parser.parse_docstring(docstring)
        print (result['description'])
(Note parse_docstrings return a dictionary)


`Parinx` is a Python module that makes working with Sphinx feel like you are working with [JSON](http://docs.python.org/library/json.html):

parse_docstring
===============

`parse_docstring` takes `docstring` and `cls` i.e. class name as argument. Default value for cls is `None`.

It returns dict which contains
    `description` of method
    `arguments` contains dict of dict: zone_id{type_name, desciption and required}
    `return` contains dict: {type_name, class, description}

When `cls=None`

```python
>>> from parinx.parser import parse_docstring
>>> docstring = """
...         Return a Zone instance.
...         Second line docsting.
...
...         :type zone_id: ``str``
...         :param zone_id: Required zone id (required)
...
...         :keyword    auth:   Initial authentication information for the node
...                             (optional)
...         :type       auth: :class:`NodeAuthSSHKey` or `NodeAuthPassword`
...
...         :return:    instance
...         :rtype: :class:`Zone` or `Node`
...         """
>>> result = parse_docstring(docstring)
>>> result
{'return': {'type_name': 'class:`Zone` or `Node`', 'description': 'instance'}, 'description': 'Return a Zone instance. Second line docsting.', 'arguments': defaultdict(<function <lambda> at 0x200be60>, {'zone_id': {'type_name': '``str``', 'description': 'Required zone id (required)', 'required': True}, 'auth': {'type_name': 'class:`NodeAuthSSHKey` or `NodeAuthPassword`', 'description': 'Initial authentication information for the node (optional)', 'required': False}})}
>>> result['arguments']
defaultdict(<function <lambda> at 0x200bed8>, {'zone_id': {'type_name': '``str``', 'description': 'Required zone id (required)', 'required': True}, 'auth': {'type_name': 'class:`NodeAuthSSHKey` or `NodeAuthPassword`', 'description': 'Initial authentication information for the node (optional)', 'required': False}})
>>> result['description']
'Return a Zone instance. Second line docsting.'
>>> return_type = result['return']['type_name']
>>> return_type
'class:`Zone` or `Node`'

When `cls!= None`
>>> class Parent(object):
...     def create_node(self, **kwargs):
...         """
...         Create a new node instance.
...
...         :keyword    name:   String with a name for this new node (required)
...         :type       name:   ``str``
...
...         :keyword    size:   The size of resources allocated to this node.
...                                 (required)
...         :type       size:   ``dict``
...
...         :return: The newly created node.
...         :rtype: :class:`Node`
...         """
...
>>> result = parse_docstring(docstring, cls=Parent)
>>> result
{'return': {'type_name': 'class:`Zone` or `Node`', 'description': 'instance'}, 'description': 'Return a Zone instance. Second line docsting.', 'arguments': defaultdict(<function <lambda> at 0x19bee60>, {'zone_id': {'type_name': '``str``', 'description': 'Required zone id (required)', 'required': True}, 'auth': {'type_name': 'class:`NodeAuthSSHKey` or `NodeAuthPassword`', 'description': 'Initial authentication information for the node ...                             (optional)', 'required': False}})}

split_docstring
===============

`split_docstring' return description and list of fields strings.

Supported field strings are:
':param'
':type'
':keyword'
':rtype:'
'@inherits:'
':return:'

```python
>>> result = split_docstring(docstring)
>>> result
('Return a node instance.', [':keyword    name:   String with a name for this new node (required)', ':type       name:   ``str`` ', ':keyword    size:   The size of resources allocated to this node. (required)', ':type       size:   ``dict`` ', ':return: instance', ':rtype: :class:`Node` '])
>>> result[1]
[':keyword    name:   String with a name for this new node (required)', ':type       name:   ``str`` ', ':keyword    size:   The size of resources allocated to this node. (required)', ':type       size:   ``dict`` ', ':return: instance', ':rtype: :class:`Node` ']
>>> result[1][3]
':type       size:   ``dict`` '


get_method_docstring
====================

`get_method_docstring` takes class name(cls) and method name as argument(method_name).

It returns method docstring
If method docstring is empty then it takes it from parent class

.. code-block:: python
class Parent(object):
    def create_node(self, **kwargs):
        """
        Create a new node instance.

        :keyword    name:   String with a name for this new node (required)
        :type       name:   ``str``

        :keyword    size:   The size of resources allocated to this node.
                                (required)
        :type       size:   ``dict``

        :return: The newly created node.
        :rtype: :class:`Node`
        """

class Child(Parent):
    def return_node(self, **kwargs):
        """
        Return a Zone instance.
        Second line docsting.

        :type zone_id: ``str``
        :param zone_id: Required zone id (required)

        :keyword    auth:   Initial authentication information for the node
                            (optional)
        :type       auth: :class:`NodeAuthSSHKey` or `NodeAuthPassword`

        :return:    instance
        :rtype: :class:`Zone` or `Node`
        """

```python
>>> from parinx.parser import get_method_string
>>> get_method_docstring(Parent, 'create_node')
'Create a new node instance.\n\n:keyword    name:   String with a name for this new node (required)\n:type       name:   ``str``\n\n:keyword    size:   The size of resources allocated to this node.\n                        (required)\n:type       size:   ``dict``\n\n:return: The newly created node.\n:rtype: :class:`Node`'
>>> get_method_docstring(Child, 'return_node')
'Return a Zone instance.\nSecond line docsting.\n\n:type zone_id: ``str``\n:param zone_id: Required zone id (required)\n\n:keyword    auth:   Initial authentication information for the node\n                    (optional)\n:type       auth: :class:`NodeAuthSSHKey` or `NodeAuthPassword`\n\n:return:    instance\n:rtype: :class:`Zone` or `Node`'
>>> get_method_docstring(Child, 'create_node')
'Create a new node instance.\n\n:keyword    name:   String with a name for this new node (required)\n:type       name:   ``str``\n\n:keyword    size:   The size of resources allocated to this node.\n                        (required)\n:type       size:   ``dict``\n\n:return: The newly created node.\n:rtype: :class:`Node`'
>>> get_method_docstring(Parent, 'return_node')

