Supported options
=================

The recipe supports the any number of options which are python functions. In fact they are actually
instance methods of the instance of the scripty recipe. Since the ini parser used with buildout
doesn't preserve initial whitespace each line of your method should start with a `...` followed
by the whitespace as per normal python. Since this is a method you can provide a "return" statement.
The return value will be stored as a value in the buildout parts options which is available for
replacement in other buildout parts.


Example usage
=============


Let's say you want to transform the a varnish:backends value to what can
be used inside haproxy::

    >>> write('buildout.cfg',
    ... """
    ... [buildout]
    ... parts = scripty echobackends echorepeat
    ...
    ... [varnish]
    ... backends =
    ... 	myhost.com:255.255.255.1
    ...     myhost2.com:125.125.125.1
    ...
    ... [scripty]
    ... recipe = mr.scripty
    ... backends =
    ...   ... res = ""
    ...   ... for line in self.buildout['varnish']['backends']:
    ...   ...    if ':' not in line:
    ...   ...      continue
    ...   ...    host,dest = line.split(':')
    ...   ...    res += "acl %s url_sub VirtualHostRoot/%s\\n" % (dest,host)
    ...   ... return res
    ... repeat =
    ...   ... opt_repeatx = int(self.options['repeatx'])
    ...   ... fun_repeatx = self.repeatx()
    ...   ... return '\\n'.join(["this is line %s"%i for i in range(1,opt_repeatx+1)])
    ... repeatx = return '10'
    ...
    ... [echobackends]
    ... recipe = mr.scripty
    ... install = print self.buildout['scripty']['backends']; return []
    ...
    ... [echorepeat]
    ... recipe = mr.scripty
    ... install =
    ...   ... script = self.buildout['scripty']
    ...   ... print script['repeat']
    ...   ... return []
    ... """)

Running the buildout gives us::

    >>> print 'start', system(buildout) 
    start...
    Installing echobackends.
    acl  url_sub VirtualHostRoot/
    acl  url_sub VirtualHostRoot/
    <BLANKLINE>
    Installing echorepeat.
    this is line 1
    this is line 2
    this is line 3
    this is line 4
    this is line 5
    this is line 6
    this is line 7
    this is line 8
    this is line 9
    this is line 10
    <BLANKLINE>

From this example you'll notice several things. Options that are part of a mr.scripty part are
turned into methods of the part instance and can call each other. In addition, each method can
be called from other buildout recipes by accessing the option via ${part:method} or in code via
self.buildout[part][method]. What is returned is a lazily evaluated string that will only call
the method once the string is actually needed.