QIf
==========
----

QIf表示量子程序条件判断操作，输入参数为条件判断表达式，功能是执行条件判断。

.. _api_introduction:

接口介绍
>>>>>>>>>>>
----

在QPanda2中，QIfProg类用于表示执行量子程序条件判断操作，初始化一个QIfProg对象有以下两种方式：

C++风格

    .. code-block:: c

        QIfProg qif = QIfProg(ClassicalCondition, QProg);
        QIfProg qif = QIfProg(ClassicalCondition, QProg, QProg);

C语言风格

    .. code-block:: c

        QIfProg qif = CreateIfProg(ClassicalCondition, QProg);
        QIfProg qif = CreateIfProg(ClassicalCondition, QProg, QProg);

        QIfProg qif = createIfProg(ClassicalCondition, QProg);
        QIfProg qif = createIfProg(ClassicalCondition, QProg, QProg);

上述函数需要提供两种类型参数，即ClassicalCondition与QProg，
当传入1个QProg参数时，QProg表示正确分支，当传入2个QProg参数时，第一个表示正确分支，第二个表示错误分支

.. note:: 由于QNode*、 shared_ptr<QNode>、QCircuit、QIfProg、QWhileProg、QGate、QMeasure、ClassicalCondition可以隐式转换为QProg，
    所以在构建QIf时第二个参数或第三个参数也可以传入上述中的任意一种节点。

同时，通过该类内置的函数可以轻松获取QIf操作正确分支与错误分支

    .. code-block:: c

        QIfProg qif = createIfProg(ClassicalCondition, QProg, QProg);
        QNode* true_branch  = qif.getTrueBranch();
        QNode* false_branch = qif.getFalseBranch();

也可以获取量子表达式

    .. code-block:: c

        QIfProg qif = createIfProg(ClassicalCondition, QProg, QProg);
        ClassicalCondition cc = qif.getCExpr();

或
    .. code-block:: c

        QIfProg qif = createIfProg(ClassicalCondition, QProg, QProg);
        ClassicalCondition cc = qif.getClassicalCondition();

实例
>>>>>>>>>
----

    .. code-block:: c

        #include "QPanda.h"
        USING_QPANDA

        int main()
        {
            init();
            QProg prog;

            auto qvec = qAllocMany(3);
            auto cvec = cAllocMany(3);
            cvec[1].set_val(0);
            cvec[0].set_val(0);

            QProg branch_true, branch_false;
            
            // 构建QIf正确分支以及错误分支
            branch_true << H(qvec[cvec[0]]) << (cvec[0]=cvec[0]+1);
            branch_false << H(qvec[0]) << CNOT(qvec[0],qvec[1]) 
                         << CNOT(qvec[1],qvec[2]);

            // 构建QIf
            auto qif = createIfProg(cvec[1]>5, branch_true, branch_false);

            // QIf插入到量子程序中
            prog << qif;

            // 概率测量，并返回目标量子比特的概率测量结果，下标为十进制
            auto result = probRunTupleList(prog, qvec);

            // 打印概率测量结果
            for (auto & val : result)
            {
                std::cout << val.first << ", " << val.second << std::endl;
            }

            finalize();
            return 0;
        }

运行结果：

    .. code-block:: c

        0, 0.5
        7, 0.5
        1, 0
        2, 0
        3, 0
        4, 0
        5, 0
        6, 0

.. warning::

    ``CreateIfProg``、 ``getCExpr`` 等在后续的版本中会被舍弃。