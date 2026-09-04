## 1. Instalação do Pacote NuGet

dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer


## 2. Configuração do Program.cs

using System.Text;
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.IdentityModel.Tokens;

var builder = WebApplication.CreateBuilder(args);

// 1. Chave secreta de autenticação (mínimo de 32 caracteres)
var secretKey = Encoding.ASCII.GetBytes("SUA_CHAVE_SUPER_SECRETA_E_LONGA_COM_32_CARACTERES!");

// 2. Configura a Autenticação por JWT
builder.Services.AddAuthentication(options =>
{
    options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
    options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
})
.AddJwtBearer(options =>
{
    options.RequireHttpsMetadata = false;
    options.SaveToken = true;
    options.TokenValidationParameters = new TokenValidationParameters
    {
        ValidateIssuerSigningKey = true,
        IssuerSigningKey = new SymmetricSecurityKey(secretKey),
        ValidateIssuer = false,
        ValidateAudience = false
    };
});

builder.Services.AddControllers();

var app = builder.Build();

// 3. Ativa os Middlewares de Segurança
app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();

app.Run();
