Код:
```
using System;

namespace PortfolioLab
{
    class Program
    {
        static void Main(string[] args)
        {
            double[] m = { 0.10, 0.30, 0.45 };
            double[] sigma = { 0.0, 0.10, 0.15 };
            double[,] rho = {
                { 1.0, 0.0, 0.0 },
                { 0.0, 1.0, -0.8 },
                { 0.0, -0.8, 1.0 }
            };

            Console.WriteLine("a) Portfolio analysis for three assets");
            Console.WriteLine();

            BuildEfficientFrontier(m, sigma, rho);

            Console.WriteLine();
            Console.WriteLine("b) Target portfolio with expected return = 25%");
            SolvePortfolioForTarget(m, sigma, rho, 0.25);

            Console.WriteLine();
            Console.WriteLine("c) Target portfolio with expected return = 40%");
            SolvePortfolioForTarget(m, sigma, rho, 0.40);

        }

        static void BuildEfficientFrontier(double[] m, double[] sigma, double[,] rho)
        {
            int nRisk = 2;
            double[,] covRisk = new double[nRisk, nRisk];
            for (int i = 0; i < nRisk; i++)
                for (int j = 0; j < nRisk; j++)
                    covRisk[i, j] = sigma[i + 1] * sigma[j + 1] * rho[i + 1, j + 1];

            int steps = 50;
            for (int k = 0; k <= steps; k++)
            {
                double w2 = k / (double)steps;
                double w3 = 1.0 - w2;
                double mean = w2 * m[1] + w3 * m[2];
                double var = w2 * w2 * covRisk[0, 0] + w3 * w3 * covRisk[1, 1] + 2 * w2 * w3 * covRisk[0, 1];
                double sd = var >= 0 ? Math.Sqrt(var) : 0.0;
                Console.WriteLine($"w2={w2:F3} w3={w3:F3}  mP={mean * 100:F3}%  sigma*P={sd * 100:F3}%");
            }
        }

        static void SolvePortfolioForTarget(double[] m, double[] sigma, double[,] rho, double targetReturn)
        {
            int n = m.Length;
            double[,] cov = new double[n, n];
            for (int i = 0; i < n; i++)
                for (int j = 0; j < n; j++)
                    cov[i, j] = sigma[i] * sigma[j] * rho[i, j];

            int dim = n + 2;
            double[,] A = new double[dim, dim];
            double[] b = new double[dim];

            for (int i = 0; i < n; i++)
                for (int j = 0; j < n; j++)
                    A[i, j] = 2.0 * cov[i, j];

            for (int i = 0; i < n; i++)
            {
                A[i, n] = -m[i];
                A[i, n + 1] = -1.0;
                A[n, i] = m[i];
                A[n + 1, i] = 1.0;
            }

            b[n] = targetReturn;
            b[n + 1] = 1.0;

            double[] sol;
            try
            {
                sol = SolveLinearSystem(A, b);
            }
            catch (ArgumentException ex)
            {
                Console.WriteLine("Linear solve failed: " + ex.Message);
                return;
            }

            double[] x = new double[n];
            Array.Copy(sol, x, n);

            double mean = 0.0;
            for (int i = 0; i < n; i++) mean += x[i] * m[i];

            double variance = 0.0;
            for (int i = 0; i < n; i++)
                for (int j = 0; j < n; j++)
                    variance += x[i] * x[j] * cov[i, j];

            double sd = variance >= 0 ? Math.Sqrt(variance) : 0.0;

            Console.WriteLine($"weights: x1={x[0]:F6}, x2={x[1]:F6}, x3={x[2]:F6}");
            Console.WriteLine($"expected return mP = {mean * 100:F4}%");
            Console.WriteLine($"d) portfolio risk sigma*P = {sd * 100:F4}%");
        }

        static double[] SolveLinearSystem(double[,] Aorig, double[] borig)
        {
            int n = borig.Length;
            double[,] A = new double[n, n];
            double[] b = new double[n];
            for (int i = 0; i < n; i++)
            {
                b[i] = borig[i];
                for (int j = 0; j < n; j++) A[i, j] = Aorig[i, j];
            }

            int[] pivot = new int[n];
            for (int i = 0; i < n; i++) pivot[i] = i;

            for (int k = 0; k < n; k++)
            {
                int maxRow = k;
                double maxVal = Math.Abs(A[k, k]);
                for (int i = k + 1; i < n; i++)
                {
                    double v = Math.Abs(A[i, k]);
                    if (v > maxVal) { maxVal = v; maxRow = i; }
                }

                if (Math.Abs(A[maxRow, k]) < 1e-12) throw new ArgumentException("Singular matrix");

                if (maxRow != k)
                {
                    for (int j = k; j < n; j++)
                    {
                        double tmp = A[k, j]; A[k, j] = A[maxRow, j]; A[maxRow, j] = tmp;
                    }
                    double tb = b[k]; b[k] = b[maxRow]; b[maxRow] = tb;
                    int tp = pivot[k]; pivot[k] = pivot[maxRow]; pivot[maxRow] = tp;
                }

                double akk = A[k, k];
                for (int j = k; j < n; j++) A[k, j] /= akk;
                b[k] /= akk;

                for (int i = 0; i < n; i++)
                {
                    if (i == k) continue;
                    double factor = A[i, k];
                    for (int j = k; j < n; j++) A[i, j] -= factor * A[k, j];
                    b[i] -= factor * b[k];
                }
            }

            return b;
        }
    }
}
```
Результат: 
<img width="609" height="499" alt="image" src="https://github.com/user-attachments/assets/a72d5aea-8a68-4336-ae7d-e139f116953e" />
