QWhile
==============
----

量子程序循环控制操作，输入参数为条件判断表达式，功能是执行while循环操作。

.. _api_introduction:

接口介绍
>>>>>>>>>>>>>
----

在QPanda2中，QWhileProg类用于表示执行量子程序while循环操作，初始化一个QWhileProg对象有以下两种方式：

C++风格

    .. code-block:: c

        QWhileProg qwile = QWhileProg(ClassicalCondition, QProg);

C语言风格

    .. code-block:: c

        QWhileProg qwile = CreateWhileProg(ClassicalCondition, QProg);

        QWhileProg qwile = createWhileProg(ClassicalCondition, QProg);


上述函数需要提供两个参数，即ClassicalCondition(量子表达式)与QProg(量子程序)

.. note:: 由于QNode*、 shared_ptr<QNode>、QCircuit、QIfProg、QWhileProg、QGate、QMeasure、ClassicalCondition可以隐式转换为QProg，
    所以在构建QWhile时第二个参数也可以传入上述中的任意一种节点。

同时，通过该类内置的函数可以轻松获取QWhile操作正确分支

    .. code-block:: c

        QWhileProg qwhile = createWhileProg(ClassicalCondition, QProg);
        QNode* true_branch = qwhile.getTrueBranch();

也可以获取量子表达式

    .. code-block:: c

        QWhileProg qwhile = createWhileProg(ClassicalCondition, QProg);
        ClassicalCondition cc = qwhile.getCExpr();

或

    .. code-block:: c

        QWhileProg qwhile = createWhileProg(ClassicalCondition, QProg);
        ClassicalCondition cc = qwhile.getClassicalCondition();

具体的操作流程可以参考下方示例

实例
>>>>>>>>>>
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
            cvec[0].set_val(0);

            // 构建QWhile的循环分支
            QProg prog_in;
            prog_in << BARRIER(qvec) << cvec[0] << H(qvec[cvec[0]]) 
                    << (cvec[0] = cvec[0]+1);
            
            // 构建QWhile
            auto qwhile = createWhileProg(cvec[0]<3, prog_in);

            // QWhile插入到量子程序中
            prog << qwhile;

            // 概率测量，并返回目标量子比特的概率测量结果，其对应的下标为十进制
            auto result = probRunTupleList(prog, qvec);

            // 打印测量结果
            for (auto & val : result)
            {
                std::cout << val.first << ", " << val.second << std::endl;
            }

            finalize();
            return 0;
        }
        
运行结果：

    .. code-block:: c

        0, 0.125
        1, 0.125
        2, 0.125
        3, 0.125
        4, 0.125
        5, 0.125
        6, 0.125
        7, 0.125

.. warning::

    ``CreateQWhile``、 ``getCExpr`` 等在后续的版本中会被舍弃。