The QueryBuilder
================

The ``QueryBuilder`` provides an API that is designed for
programmatically constructing a query in several steps.

It provides a set of classes and methods that is able to
programmatically build queries, and also provides a fluent API.

The QueryBuilder of PHPCR-ODM is provided by the phpcr-utils library.
See `PHPCR-utils > Namespaces > PHPCR\Util\QOM\QueryBuilder <http://phpcr.github.com/doc/html-all/index.html>`_
for a full documentation on the QueryBuilder methods.

The actual tests and such are done by the PHPCR Query Object Model factory,
defined by ``PHPCR\Query\QOM\QueryObjectModelFactoryInterface``

.. contents::

Constructing a new QueryBuilder object
--------------------------------------

The same way you build a normal Query, you build a ``QueryBuilder``
object, with a factory method on the DocumentManager:

.. code-block:: php

    <?php
    /** @var $dm DocumentManager */

    $qb = $dm->createQueryBuilder();

Once you have created an instance of QueryBuilder, it provides you
with the methods to build a query.

Working with QueryBuilder
-------------------------

This query is equivalent to the JCR-SQL2 query ``SELECT * FROM nt:unstructured WHERE name NOT IS NULL``

.. code-block:: php

    <?php

    /** @var $qb QueryBuilder */
    $factory = $qb->getQOMFactory();
    $qb->from($factory->selector('nt:unstructured'))
        ->where($factory->propertyExistence('name'))
        ->execute();

    $result = $documentManager->getDocumentsByQuery($qb->getQuery());
    foreach ($result as $document) {
        echo $document->getId();
    }

The maximum number of results (limit) can be set with the setMaxResults method.
Furthermore the position of the first result to be retrieved (offset) can be
set with setFirstResult

.. code-block:: php

    <?php

    /** @var $qb QueryBuilder */
    $factory = $qb->getQOMFactory();
    $qb->from($factory->selector('nt:unstructured'))
        ->where($factory->propertyExistence('name'))
        ->setFirstResult(5)
        ->setMaxResults(10)
        ->execute();

Getting all descendant nodes of /dms is as simple as adding a descendant node constraint:

.. code-block:: php

    <?php

    /** @var $qb QueryBuilder */
    $factory = $qb->getQOMFactory();
    $qb->from($factory->selector('nt:unstructured'))
        ->where($factory->descendantNode('/dms'))
        ->execute();

Note that if you just need the direct children of a document, you should use
the ``@Children`` annotation on the document.

If you want to know the SQL2 statement generated call getStatement() on the query object.

.. code-block:: php

    <?php
    //Prepare the query builder with the desired statement.
    //..
    echo $qb->getQuery()->getStatement();

Helper methods
--------------

.. _querybuilderref_addorderby:

addOrderBy
~~~~~~~~~~

Adds an ordering to the query results

Arguments:

-  **sort**: The ordering expression. Instance of `DynamicOperandInterface`.
-  **order**: The ordering direction, one of ``ASC`` or ``DESC``

.. code-block:: php

   <?php
   $qb->addOrderBy($qb->qomf()->propertyValue('date'), 'ASC');

.. _querybuilderref_addselect:

addSelect
~~~~~~~~~

@todo

Adds a property in the specified or default selector to include in
the tabular view of the query results.

Arguments:

- **propertyName**: Name of the property
- **columnName**: ??
- **selectorName**: ??

.. code-block:: php

   <?php
   $qb->addSelect(#,#,#);

.. _querybuilderref_andWhere:
   
andWhere
~~~~~~~~

Creates a new constraint formed by applying a logical AND to the
existing constraint and the new one

Arguments:

- **constraint**: Constraint, instance of ``ConstraintInterface``

.. code-block::

    <?php
    use PHPCR\Query\QOM\QueryObjectModelConstantsInterface as QOMConstants;

    $qb->andWhere($qb->qomf()->comparison(
        $qb->qomf()->propertyValue('blog'),
        QOMConstants::JCR_OPERATOR_EQUAL_TO,
        $qb->qomf()->literal('My Blog')
    ));

.. _querybuilderref_from:

from
~~~~

Sets the default Selector or the node-tuple source. Can be a selector
or a join.

Arguments:

- **source**: Source, instance of ``SourceInterface``.

.. code-block:: php

    <?php
    $qb->from($qb->qomf()->selector('nt:unstructured'));

.. _querybuilderref_innerjoin:
   
innerJoin
~~~~~~~~~

@todo

Performs an inner join between the stored source and the supplied source.

Arguments:

- **rightSource**: Selector for table to join, instance of ``SourceInterface``.
- **joinCondition**: Join condition, instance of ``JoinConditionInterface``.

.. code-block:: php

    <?php
    $qb->innerJoin(
        $qb->qomf()->selector('my:nodetype1'),
        $qb->qomf()->equiJoinCondition(...)
    );
    
.. _querybuilderref_join:
   
join
~~~~

@todo

Performs an join between the stored source and the supplied source.

Arguments:

- **rightSource**: Selector for table to join, instance of ``SourceInterface``.
- **joinCondition**: Join condition, instance of ``JoinConditionInterface``.

.. code-block:: php

    <?php
    $qb->join(
        $qb->qomf()->selector('my:nodetype1'),
        $qb->qomf()->equiJoinCondition(...)
    );
  
.. _querybuilderref_joinwithtype:
   
joinWithType
~~~~~~~~~~~~

Performs a join between the stored source and the supplied source with the
specified join type.

Arguments:

- **rightSource**: Selector for table to join, instance of ``SourceInterface``.
- **joinType**: One of the constants ``JCR_JOIN_TYPE_INNER``, ``JCR_JOIN_TYPE_LEFT_OUTER`` 
  or ``JCR_JOIN_TYPE_RIGHT_OUTER`` defined in the ``QueryObjectModelConstantsInterface``.
- **joinCondition**: Join condition, instance of ``JoinConditionInterface``.

.. code-block:: php

    <?php
    use PHPCR\Query\QOM\QueryObjectModelConstantsInterface as QOMConstants;

    $qb->joinWithType(
        $qb->qomf()->selector('my:nodetype1'),
        QOMConstants::JCR_JOIN_TYPE_INNER,
        $qb->qomf()->equiJoinCondition(...)
    );

.. _querybuilderref_leftjoin:
   
leftJoin
~~~~~~~~

Performs an left outer join between the stored source and the supplied source.

Arguments:

- **rightSource**: Selector for table to join, instance of ``SourceInterface``.
- **joinCondition**: Join condition, instance of ``JoinConditionInterface``.

.. code-block:: php

    <?php
    $qb->leftJoin(
        $qb->qomf()->selector('my:nodetype1'),
        $qb->qomf()->equiJoinCondition(...)
    );

.. _querybuilderref_orwhere:
   
orWhere
~~~~~~~

Creates a new constraint formed by applying a logical OR to the
existing constraint and the new one. Order is important.

Arguments:

- **constraint**: Constraint, instance of ``ConstraintInterface``

.. code-block::

    <?php
    use PHPCR\Query\QOM\QueryObjectModelConstantsInterface as QOMConstants;

    $qb->orWhere($qb->qomf()->comparison(
        $qb->qomf()->propertyValue('blog'),
        QOMConstants::JCR_OPERATOR_EQUAL_TO,
        $qb->qomf()->literal('My Blog')
    ));

.. _querybuilderref_orderby:
   
orderBy
~~~~~~~

Sets ordering to the query results, replacing any existing orderings.

Arguments:

-  **sort**: The ordering expression. Instance of `DynamicOperandInterface`.
-  **order**: The ordering direction, one of ``ASC`` or ``DESC``

.. code-block:: php

   <?php
   $qb->orderBy($qb->qomf()->propertyValue('date'), 'ASC');

.. _querybuilderref_rightjoin:
   
rightJoin
~~~~~~~~~

Performs an right outer join between the stored source and the supplied source.

Arguments:

- **rightSource**: Selector for table to join, instance of ``SourceInterface``.
- **joinCondition**: Join condition, instance of ``JoinConditionInterface``.

.. code-block:: php

    <?php
    $qb->rightJoin(
        $qb->qomf()->selector('my:nodetype1'),
        $qb->qomf()->equiJoinCondition(...)
    );


.. _querybuilderref_select:
   
select
~~~~~~

@todo:

Identifies a property in the specified or default selector to include in
the tabular view of the query results. Replaces any previously specified
columns to be selected, if any.

Arguments:

- **propertyName**: Name of the property
- **columnName**: ??
- **selectorName**: ??

.. code-block:: php

   <?php
   $qb->select(#,#,#);

.. _querybuilderref_setcolumns:
   
setColumns
~~~~~~~~~~

@todo

Sets the columns to be selected.

Arguments:

- **columns**: Array of ???

.. code-block:: php

    <?php
    $qb->setColumns(array(
        ...,
        ...,
        ...,
    );

.. _querybuilderref_where:
   
where
~~~~~

Specifies one restriction (may be simple or composed).
Replaces any previously specified restrictions, if any.

Arguments:

- **constraint**: Constraint, instance of ``ConstraintInterface``

.. code-block::

    <?php
    use PHPCR\Query\QOM\QueryObjectModelConstantsInterface as QOMConstants;

    $qb->where($qb->qomf()->comparison(
        $qb->qomf()->propertyValue('blog'),
        QOMConstants::JCR_OPERATOR_EQUAL_TO,
        $qb->qomf()->literal('My Blog')
    ));
