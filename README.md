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

## 3. Modelo DTO (LoginDto.cs)

    namespace SuaApi.DTOs;
    
    public record LoginDto(string Email, string Senha);

## 4. Controller de Autenticação (AuthController.cs)

    using System.IdentityModel.Tokens.Jwt;
    using System.Security.Claims;
    using System.Text;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.IdentityModel.Tokens;
    using SuaApi.DTOs;
    
    namespace SuaApi.Controllers;
    
    [ApiController]
    [Route("api/[controller]")]
    public class AuthController : ControllerBase
    {
        [HttpPost("login")]
        public IActionResult Login([FromBody] LoginDto dto)
        {
            if (dto.Email != "admin@email.com" || dto.Senha != "123456")
            {
                return Unauthorized(new { mensagem = "E-mail ou senha inválidos." });
            }
    
            var tokenHandler = new JwtSecurityTokenHandler();
            var key = Encoding.ASCII.GetBytes("SUA_CHAVE_SUPER_SECRETA_E_LONGA_COM_32_CARACTERES!");
            
            var tokenDescriptor = new SecurityTokenDescriptor
            {
                Subject = new ClaimsIdentity(new[]
                {
                    new Claim(ClaimTypes.Name, dto.Email)
                }),
                Expires = DateTime.UtcNow.AddHours(2),
                SigningCredentials = new SigningCredentials(
                    new SymmetricSecurityKey(key), 
                    SecurityAlgorithms.HmacSha256Signature)
            };
    
            var token = tokenHandler.CreateToken(tokenDescriptor);
    
            return Ok(new { token = tokenHandler.WriteToken(token) });
        }
    }

## 5. Controller Protegida (TarefasController.cs)

    using Microsoft.AspNetCore.Authorization;
    using Microsoft.AspNetCore.Mvc;
    
    namespace SuaApi.Controllers;
    
    [ApiController]
    [Route("api/[controller]")]
    [Authorize]
    public class TarefasController : ControllerBase
    {
        [HttpGet]
        public IActionResult Listar()
        {
            return Ok(new[] 
            { 
                new { id = 1, titulo = "Estudar para o SAEP" },
                new { id = 2, titulo = "Testar rotas no Thunder Client" }
            });
        }
    }

## 6. Instruções de Teste (Thunder Client / Postman)

app.MapControllers();

app.Run();
