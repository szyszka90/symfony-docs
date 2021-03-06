Callback
========

The purpose of the Callback constraint is to create completely custom
validation rules and to assign any validation errors to specific fields
on your object. If you're using validation with forms, this means that you
can make these custom errors display next to a specific field, instead of
simply at the top of your form.

This process works by specifying one or more *callback* methods, each of
which will be called during the validation process. Each of those methods
can do anything, including creating and assigning validation errors.

.. note::

    A callback method itself doesn't *fail* or return any value. Instead,
    as you'll see in the example, a callback method has the ability to directly
    add validator "violations".

+----------------+------------------------------------------------------------------------+
| Applies to     | :ref:`class <validation-class-target>`                                 |
+----------------+------------------------------------------------------------------------+
| Options        | - :ref:`callback <callback-option>`                                    |
|                | - `payload`_                                                           |
+----------------+------------------------------------------------------------------------+
| Class          | :class:`Symfony\\Component\\Validator\\Constraints\\Callback`          |
+----------------+------------------------------------------------------------------------+
| Validator      | :class:`Symfony\\Component\\Validator\\Constraints\\CallbackValidator` |
+----------------+------------------------------------------------------------------------+

Configuration
-------------

.. configuration-block::

    .. code-block:: php-annotations

        // src/AppBundle/Entity/Author.php
        namespace AppBundle\Entity;

        use Symfony\Component\Validator\Constraints as Assert;
        use Symfony\Component\Validator\Context\ExecutionContextInterface;
        // if you're using the older 2.4 validation API, you'll need this instead
        // use Symfony\Component\Validator\ExecutionContextInterface;

        class Author
        {
            /**
             * @Assert\Callback
             */
            public function validate(ExecutionContextInterface $context)
            {
                // ...
            }
        }

    .. code-block:: yaml

        # src/AppBundle/Resources/config/validation.yml
        AppBundle\Entity\Author:
            constraints:
                - Callback: [validate]

    .. code-block:: xml

        <!-- src/AppBundle/Resources/config/validation.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>
        <constraint-mapping xmlns="http://symfony.com/schema/dic/constraint-mapping"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/dic/constraint-mapping http://symfony.com/schema/dic/constraint-mapping/constraint-mapping-1.0.xsd">

            <class name="AppBundle\Entity\Author">
                <constraint name="Callback">validate</constraint>
            </class>
        </constraint-mapping>

    .. code-block:: php

        // src/AppBundle/Entity/Author.php
        namespace AppBundle\Entity;

        use Symfony\Component\Validator\Mapping\ClassMetadata;
        use Symfony\Component\Validator\Constraints as Assert;

        class Author
        {
            public static function loadValidatorMetadata(ClassMetadata $metadata)
            {
                $metadata->addConstraint(new Assert\Callback('validate'));
            }
        }

The Callback Method
-------------------

The callback method is passed a special ``ExecutionContextInterface`` object.
You can set "violations" directly on this object and determine to which
field those errors should be attributed::

    // ...
    use Symfony\Component\Validator\Context\ExecutionContextInterface;
    // if you're using the older 2.4 validation API, you'll need this instead
    // use Symfony\Component\Validator\ExecutionContextInterface;

    class Author
    {
        // ...
        private $firstName;

        public function validate(ExecutionContextInterface $context)
        {
            // somehow you have an array of "fake names"
            $fakeNames = array(/* ... */);

            // check if the name is actually a fake name
            if (in_array($this->getFirstName(), $fakeNames)) {
                // If you're using the new 2.5 validation API (you probably are!)
                $context->buildViolation('This name sounds totally fake!')
                    ->atPath('firstName')
                    ->addViolation();

                // If you're using the old 2.4 validation API
                /*
                $context->addViolationAt(
                    'firstName',
                    'This name sounds totally fake!'
                );
                */
            }
        }
    }

Static Callbacks
----------------

You can also use the constraint with static methods. Since static methods don't
have access to the object instance, they receive the object as the first argument::

    public static function validate($object, ExecutionContextInterface $context)
    {
        // somehow you have an array of "fake names"
        $fakeNames = array(/* ... */);

        // check if the name is actually a fake name
        if (in_array($object->getFirstName(), $fakeNames)) {
            // If you're using the new 2.5 validation API (you probably are!)
            $context->buildViolation('This name sounds totally fake!')
                ->atPath('firstName')
                ->addViolation()
            ;

            // If you're using the old 2.4 validation API
            $context->addViolationAt(
                'firstName',
                'This name sounds totally fake!'
            );
        }
    }

External Callbacks and Closures
-------------------------------

If you want to execute a static callback method that is not located in the
class of the validated object, you can configure the constraint to invoke
an array callable as supported by PHP's :phpfunction:`call_user_func` function.
Suppose your validation function is ``Vendor\Package\Validator::validate()``::

    namespace Vendor\Package;

    use Symfony\Component\Validator\Context\ExecutionContextInterface;
    // if you're using the older 2.4 validation API, you'll need this instead
    // use Symfony\Component\Validator\ExecutionContextInterface;

    class Validator
    {
        public static function validate($object, ExecutionContextInterface $context)
        {
            // ...
        }
    }

You can then use the following configuration to invoke this validator:

.. configuration-block::

    .. code-block:: php-annotations

        // src/AppBundle/Entity/Author.php
        namespace Acme\BlogBundle\Entity;

        use Symfony\Component\Validator\Constraints as Assert;

        /**
         * @Assert\Callback({"AppBundle\MyStaticValidatorClass", "isAuthorValid"})
         */
        class Author
        {
        }

    .. code-block:: yaml

	# src/AppBundle/Resources/config/validation.yml
	AppBundle\Entity\Author:
	    constraints:
		- Callback:
		    methods:
			-    [AppBundle\MyStaticValidatorClass, isAuthorValid]

    .. code-block:: xml

	<!-- src/AppBundle/Resources/config/validation.xml -->
	<?xml version="1.0" encoding="UTF-8" ?>
	<constraint-mapping xmlns="http://symfony.com/schema/dic/constraint-mapping"
	    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	    xsi:schemaLocation="http://symfony.com/schema/dic/constraint-mapping http://symfony.com/schema/dic/constraint-mapping/constraint-mapping-1.0.xsd">

	    <class name="AppBundle\Entity\Author">
		<constraint name="Callback">
		    <option name="methods">
			<value>
			    <value>AppBundle\MyStaticValidatorClass</value>
			    <value>isAuthorValid</value>
			</value>
		    </option>
		</constraint>
	    </class>
	</constraint-mapping>

    .. code-block:: php

	// src/AppBundle/Entity/Author.php
        namespace AppBundle\Entity;

        use Symfony\Component\Validator\Mapping\ClassMetadata;
        use Symfony\Component\Validator\Constraints as Assert;

        class Author
        {
	    public $name;

            public static function loadValidatorMetadata(ClassMetadata $metadata)
            {
		$metadata->addConstraint(new Callback(array(
		    'methods' => array(
			array(
			    'AppBundle\MyStaticValidatorClass',
			    'isAuthorValid',
			),
		    ),
		)));
            }
        }

.. note::

    The Callback constraint does *not* support global callback functions
    nor is it possible to specify a global function or a :term:`service` method
    as callback. To validate using a service, you should
    :doc:`create a custom validation constraint </cookbook/validation/custom_constraint>`
    and add that new constraint to your class.

When configuring the constraint via PHP, you can also pass a closure to the
constructor of the Callback constraint::

    // src/AppBundle/Entity/Author.php
    namespace AppBundle\Entity;

    use Symfony\Component\Validator\ExecutionContextInterface;
    use AppBundle\Entity\Author;

    use Symfony\Component\Validator\Mapping\ClassMetadata;
    use Symfony\Component\Validator\Constraints as Assert;

    class Author
    {
        public static function loadValidatorMetadata(ClassMetadata $metadata)
        {
            $callback = function ($object, ExecutionContextInterface $context) {
                // ...
            };

            $metadata->addConstraint(new Assert\Callback($callback));
        }
    }

Options
-------

.. _callback-option:

callback
~~~~~~~~

**type**: ``string``, ``array`` or ``Closure`` [:ref:`default option <validation-default-option>`]

The callback option accepts three different formats for specifying the
callback method:

* A **string** containing the name of a concrete or static method;

* An array callable with the format ``array('<Class>', '<method>')``;

* A closure.

Concrete callbacks receive an :class:`Symfony\\Component\\Validator\\Context\\ExecutionContextInterface`
instance as only argument.

Static or closure callbacks receive the validated object as the first argument
and the :class:`Symfony\\Component\\Validator\\ExecutionContextInterface`
instance as the second argument.

.. include:: /reference/constraints/_payload-option.rst.inc
