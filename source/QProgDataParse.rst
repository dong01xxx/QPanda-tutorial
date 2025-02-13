.. _QProgDataParse:

解析量子程序二进制文件
==========================

简介
--------------

解析 :ref:`QProgStored` 方法序列化的量子程序。

接口介绍
--------------

``QProgDataParse`` 类是对 :ref:`QProgStored` 类序列化的量子程序反序列化， 例如下面的量子程序：

    .. code-block:: c

        prog << H(qubits[0]) << CNOT(qubits[0], qubits[1])
                << CNOT(qubits[1], qubits[2])
                << CNOT(qubits[2], qubits[3]);

序列化之后经过base64编码之后得到的结果是（具体序列化的方法参照 :ref:`QProgStored`）

    .. code-block:: c

        AAAAAAQAAAAEAAAABAAAAA4AAQAAAAAAJAACAAAAAQAkAAMAAQACACQABAACAAMA

现在就对这个结果反序列化，先讲base64的结果解码成二进制数据：

    .. code-block:: c

        std::string = "AAAAAAQAAAAEAAAABAAAAA4AAQAAAAAAJAACAAAAAQAkAAMAAQACACQABAACAAMA";
        auto data = Base64::decode(data_str.data(), data_str.size());

然后调用 ``QProgDataParse`` 类中的函数， 得到反序列化之后的量子程序

    .. code-block:: c

        QProgDataParse parse(qvm);
        parse.load(data);
        parse.parse(parseProg);
        auto qubits_parse = parse.getQubits();
        auto cbits_parse = parse.getCbits();  

我们还可以使用QPanda2封装的一个接口：

    .. code-block:: c

        QVec qubits_parse;
        std::vector<ClassicalCondition> cbits_parse;
        convert_binary_data_to_qprog(qvm, data, qubits_parse, cbits_parse, parseProg);

实例
------------

    .. code-block:: c
    
        #include "QPanda.h"

        USING_QPANDA

        int main()
        {
            auto qvm = initQuantumMachine();
            QProg parseProg;
            QVec qubits_parse;
            std::vector<ClassicalCondition> cbits_parse;

            std::string data_str = "AAAAAAQAAAAEAAAABAAAAA4AAQAAAAAAJAACAAAAAQAkAAMAAQACACQABAACAAMA";
      
            // base64的方式解码，得到的二进制数据
            auto data = Base64::decode(data_str.data(), data_str.size());

            // 解析二进制数据，得到量子程序
            convert_binary_data_to_qprog(qvm, data, qubits_parse, cbits_parse, parseProg);

            // 概率测量，并返回目标量子比特的概率测量结果，下标为十进制
            auto result_parse = probRunTupleList(parseProg, qubits_parse);

            // 打印测量结果
            for (auto &val : result_parse)
            {
                std::cout << val.first << ", " << val.second << std::endl;
            }

            destroyQuantumMachine(qvm);
            return 0;
        }

运行结果：

    .. code-block:: c

        0, 0.5
        15, 0.5
        1, 0
        2, 0
        3, 0
        4, 0
        5, 0
        6, 0
        7, 0
        8, 0
        9, 0
        10, 0
        11, 0
        12, 0
        13, 0
        14, 0

.. note:: 可以运行出正确的结果说明可以将序列化的量子程序正确的解析出来


.. warning:: 
        新增接口 ``convert_binary_data_to_qprog()`` ，与老版本接口 ``transformBinaryDataToQProg()`` 功能相同。

