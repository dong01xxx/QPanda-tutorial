酉矩阵分解
=====================
目前，量子计算的算法通常用量子线路表示，量子线路包括量子逻辑门操作。
通常，连续的一段量子线路通常包含几十上百个甚至成千上万个量子逻辑门操作，而量子逻辑门数量或单个量子逻辑门操作的量子比特数越多，计算过程越为复杂，导致量子线路的模拟效率较低，且对硬件资源的占用较多。

算法目标
>>>>>>>>>>
----

对于上述问题，有必要对量子线路进行一种等价转换，需要减少量子线路中逻辑门的数量，
同时在此基础上，需要确保转换前后整个量子线路对应的酉矩阵完全相同.

算法概述
>>>>>>>>>>
----

本文介绍的算法是将一个N阶酉矩阵，分解成不超过r = N(N−1)/2个带有少量控制的单量子逻辑门序列，其中N=2^n，分解的产物满足如下等式关系

.. centered:: :math:`U_rU_{r-1}···U_3U_2U_1U=I_N`

从而可以得到原矩阵U的分解结果表示

.. centered:: :math:`U=U_1^{\dagger}U_2^{\dagger}U_3^{\dagger}···U_{r-1}^{\dagger}U_r^{\dagger}`

使用介绍
>>>>>>>>>>>>>>>>
----

QPanda2中设计了 ``matrix_decompose_qr`` 接口用于进行酉矩阵分解，该接口需要两个参数，
第一个是使用到的所有量子比特，第二个是待分解的酉矩阵，该函数的输出是转换后的量子线路。

实例
>>>>>>>>>>
----

.. _酉矩阵分解示例程序:
以下示例展示了部分振幅量子虚拟机接口的使用方式

.. code-block:: c
  
    #include "QPanda.h"
    USING_QPANDA

        int main()
        {
            auto qvm = new CPUQVM();
            qvm->init();
            auto q = qvm->qAllocMany(2);

            QStat source_matrix =
            {
                qcomplex_t(0.6477054522122977, 0.1195417767870219),qcomplex_t(-0.16162176706189357, -0.4020495632468249),qcomplex_t(-0.19991615329121998, -0.3764618308248643),qcomplex_t(-0.2599957197928922, -0.35935248873007863),
                qcomplex_t(-0.16162176706189363, -0.40204956324682495),qcomplex_t(0.7303014482204584, -0.4215172444390785),qcomplex_t(-0.15199187936216693, 0.09733585496768032),qcomplex_t(-0.22248203136345918, -0.1383600597660744),
                qcomplex_t(-0.19991615329122003 , -0.3764618308248644),qcomplex_t(-0.15199187936216688, 0.09733585496768032),qcomplex_t(0.6826630277354306, -0.37517063774206166),qcomplex_t(-0.3078966462928956, -0.2900897445133085),
                qcomplex_t(-0.2599957197928923, -0.3593524887300787),qcomplex_t(-0.22248203136345912, -0.1383600597660744),qcomplex_t(-0.30789664629289554, -0.2900897445133085),qcomplex_t(0.6640994547408099, -0.338593803336005)
            };

            std::cout << "source matrix:" << std::endl << source_matrix << std::endl;

        QCircuit out_cir = matrix_decompose_qr(q, source_matrix);
        auto circuit_matrix = getCircuitMatrix(out_cir);

            std::cout << "the decomposed matrix:" << std::endl << circuit_matrix << std::endl;

            if (!mat_compare(source_matrix, circuit_matrix, 0.000001))
            {
                std::cout << "matrix decompose ok !" << std::endl;
            }
            return 0;
        }
        return 0;
    }

上述实例运行的结果如下：

    .. code-block:: c

      source matrix:

      (0.647705452212298, 0.119541776787022)  (-0.161621767061894, -0.402049563246825)   (-0.19991615329122, -0.376461830824864)  (-0.259995719792892, -0.359352488730079)
      (-0.161621767061894, -0.402049563246825)   (0.730301448220458, -0.421517244439079)  (-0.151991879362167, 0.0973358549676803)  (-0.222482031363459, -0.138360059766074)
      (-0.19991615329122, -0.376461830824864)  (-0.151991879362167, 0.0973358549676803)   (0.682663027735431, -0.375170637742062)  (-0.307896646292896, -0.290089744513308)
      (-0.259995719792892, -0.359352488730079)  (-0.222482031363459, -0.138360059766074)  (-0.307896646292896, -0.290089744513308)    (0.66409945474081, -0.338593803336005)


      the decomposed matrix:

      (0.647705452212298, 0.119541776787022)  (-0.161621767061894, -0.402049563246825)   (-0.19991615329122, -0.376461830824865)  (-0.259995719792892, -0.359352488730079)
      (-0.161621767061894, -0.402049563246825)   (0.730301448220459, -0.421517244439079)  (-0.151991879362167, 0.0973358549676799)  (-0.222482031363459, -0.138360059766075)
      (-0.19991615329122, -0.376461830824865)  (-0.151991879362167, 0.0973358549676804)   (0.682663027735431, -0.375170637742062)  (-0.307896646292896, -0.290089744513309)
      (-0.259995719792892, -0.359352488730079)  (-0.222482031363459, -0.138360059766074)  (-0.307896646292896, -0.290089744513308)    (0.66409945474081, -0.338593803336005)


matrix decompose ok !

从输出的结果可以看出，分解前后的矩阵完全相同，对于一个量子比特数目确定的量子系统，
即使分解前的量子线路含有成千上万个量子逻辑门，该接口可以将分解后的量子线路复杂度控制在合理范围之内，
完全不受到分解前量子线路复杂度的影响。

    .. note::

        1. 该接口的输入参数必须为酉矩阵。
        2. 通过将分解的结果数量约束在一个限定范围内，有效减少了量子线路中的量子逻辑门数量，极大地提升了量子算法的模拟效率
        3. 示例程序中， ``getCircuitMatrix`` 接口用于获取一个量子线路对应的矩阵， ``mat_compare`` 接口用于对比两个矩阵是否完全相同（在设定的精度范围之内）