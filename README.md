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


More about dictionary
=====================

return dict have
        description - method description
        arguments - dict of dicts arg_name: {description, type_name, required}
        return - dict: {description, type}

A Sub-Section
-------------
your can query according to it.
