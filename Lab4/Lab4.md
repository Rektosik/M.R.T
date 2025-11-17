Код:
```
using System;
using MathNet.Numerics.LinearAlgebra;
using MathNet.Numerics.LinearAlgebra.Double;

class Program
{
    static void Main()
    {
        double[] mArr = { 0.10, 0.30, 0.45 };
        double[] sigma = { 0.0, 0.10, 0.15 };
        double[,] rho = {
            { 1.0, 0.0, 0.0 },
            { 0.0, 1.0, -0.8 },
            { 0.0, -0.8, 1.0 }
        };

        var m = Vector.Build.DenseOfArray(mArr);
        int n = mArr.Length;

        var cov = Matrix.Build.Dense(n, n);
        for (int i = 0; i < n; i++)
            for (int j = 0; j < n; j++)
                cov[i, j] = sigma[i] * sigma[j] * rho[i, j];

        Console.WriteLine("a) Efficient frontier (search across target returns)");
        double minR = m.Min();
        double maxR = m.Max();
        int steps = 21;
        for (int k = 0; k < steps; k++)
        {
            double target = minR + (maxR - minR) * k / (steps - 1);
            var sol = SolveForTarget(cov, m, target);
            if (sol.weights == null) continue;
            Console.WriteLine($"target={target * 100:F2}% -> mean={sol.mean * 100:F4}% sigma={sol.sd * 100:F4}%");
        }

        Console.WriteLine();
        Console.WriteLine("b) Investor's portfolio with expected return = 25%");
        var r25 = SolveForTarget(cov, m, 0.25);
        if (r25.weights == null) Console.WriteLine("No feasible solution for 25%");
        else
        {
            PrintSolution(r25);
        }

        Console.WriteLine();
        Console.WriteLine("c) Portfolios with expected return = 40%");
        var r40 = SolveForTarget(cov, m, 0.40);
        if (r40.weights == null) Console.WriteLine("No feasible solution for 40%");
        else
        {
            PrintSolution(r40);
        }

        Console.WriteLine();
        Console.WriteLine("d) Individual asset risks (std dev):");
        for (int i = 0; i < n; i++) Console.WriteLine($"sigma{i + 1} = {sigma[i] * 100:F2}%");
    }

    static (Vector<double> weights, double mean, double sd) SolveForTarget(Matrix<double> cov, Vector<double> m, double target)
    {
        int n = m.Count;
        var Iones = Vector.Build.Dense(n, 1.0);
        int dim = n + 2;
        var A = Matrix.Build.Dense(dim, dim, 0.0);
        var b = Vector.Build.Dense(dim, 0.0);
        var twoCov = cov.Multiply(2.0);
        for (int i = 0; i < n; i++)
            for (int j = 0; j < n; j++)
                A[i, j] = twoCov[i, j];
        for (int i = 0; i < n; i++)
        {
            A[i, n] = -m[i];
            A[i, n + 1] = -1.0;
            A[n, i] = m[i];
            A[n + 1, i] = 1.0;
        }
        b[n] = target;
        b[n + 1] = 1.0;
        Vector<double> sol;
        try
        {
            sol = A.Solve(b);
        }
        catch
        {
            return (null, 0.0, 0.0);
        }
        var x = sol.SubVector(0, n);
        double mean = x.DotProduct(m);
        double var = x * (cov * x);
        double sd = var >= 0 ? Math.Sqrt(var) : 0.0;
        return (x, mean, sd);
    }

    static void PrintSolution((Vector<double> weights, double mean, double sd) s)
    {
        var w = s.weights;
        Console.Write("weights: ");
        for (int i = 0; i < w.Count; i++) Console.Write($"x{i + 1}={w[i]:F8} ");
        Console.WriteLine();
        Console.WriteLine($"expected return mP = {s.mean * 100:F6}%");
        Console.WriteLine($"portfolio risk sigma*P = {s.sd * 100:F6}%");
    }
}
```
Результат: 
<img width="609" height="499" alt="image" src="https://github.com/user-attachments/assets/a72d5aea-8a68-4336-ae7d-e139f116953e" />
